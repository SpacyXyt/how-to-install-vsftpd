# 04B - Utilisateurs virtuels avec vsftpd

Cette configuration permet de créer des comptes FTP qui ne sont pas des
utilisateurs Linux. Tous les comptes virtuels sont exécutés par un compte
Linux technique nommé `ftpvirtual`.

Exemple :

```text
client1     utilisateur FTP uniquement
client2     utilisateur FTP uniquement
ftpvirtual  utilisateur Linux technique
```

> FTP transmet les identifiants en clair. En production, activez TLS/FTPS ou
> utilisez SFTP lorsque c'est possible.

## 1. Installer les paquets

```bash
sudo apt update
sudo apt install -y vsftpd libpam-pwdfile openssl
```

`openssl` sert ici a produire des hashes Unix SHA-512 (`$6$`) compatibles avec
`pam_pwdfile`. N'utilisez pas les hashes Apache APR1 (`$apr1$`) : ils ne sont
pas pris en charge par la version actuelle de `libpam-pwdfile` sur Debian 13.

## 2. Creer l'utilisateur Linux technique

```bash
sudo useradd --system --home-dir /srv/ftp --shell /usr/sbin/nologin ftpvirtual
sudo mkdir -p /srv/ftp
sudo chown root:root /srv/ftp
sudo chmod 755 /srv/ftp
id ftpvirtual
```

Le compte `ftpvirtual` ne sert jamais a se connecter directement.

## 3. Creer les utilisateurs virtuels

Creer le fichier protege :

```bash
sudo install -d -o root -g root -m 755 /etc/vsftpd
sudo touch /etc/vsftpd/virtual_users
sudo chown root:root /etc/vsftpd/virtual_users
sudo chmod 600 /etc/vsftpd/virtual_users
```

Ajouter `client1` :

Cette commande suppose que `client1` n’est pas deja present. Verifier avec :

```bash
sudo grep -q "^client1:" /etc/vsftpd/virtual_users && echo "client1 existe deja"
```

```bash
read -rsp 'Mot de passe FTP : ' FTP_PASSWORD; echo
HASH=$(printf '%s' "$FTP_PASSWORD" | openssl passwd -6 -stdin)
unset FTP_PASSWORD
printf 'client1:%s\n' "$HASH" | sudo tee -a /etc/vsftpd/virtual_users >/dev/null
unset HASH
```

Reprendre ces commandes en remplaçant `client1` pour ajouter d'autres comptes.
Le fichier doit contenir une seule ligne par compte :

```text
client1:$6$...
client2:$6$...
```

Verifier les noms sans afficher les hashes :

```bash
sudo cut -d: -f1 /etc/vsftpd/virtual_users
```

## 4. Configurer PAM

Creer `/etc/pam.d/vsftpd-virtual` :

```pam
auth required pam_pwdfile.so pwdfile /etc/vsftpd/virtual_users
account required pam_permit.so
```

Commandes equivalentes :

```bash
printf '%s\n' \
  'auth required pam_pwdfile.so pwdfile /etc/vsftpd/virtual_users' \
  'account required pam_permit.so' \
  | sudo tee /etc/pam.d/vsftpd-virtual >/dev/null
sudo chmod 644 /etc/pam.d/vsftpd-virtual
```

## 5. Configurer vsftpd

Sauvegarder la configuration actuelle :

```bash
sudo cp -a /etc/vsftpd.conf /etc/vsftpd.conf.backup
```

Remplacer `/etc/vsftpd.conf` par :

```conf
listen=YES
listen_ipv6=NO

anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022

guest_enable=YES
guest_username=ftpvirtual
virtual_use_local_privs=YES

pam_service_name=vsftpd-virtual

chroot_local_user=YES
hide_ids=YES

user_sub_token=$USER
local_root=/srv/ftp/$USER

pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

xferlog_enable=YES
log_ftp_protocol=YES
```

Avec cette configuration, `client1` arrive dans `/srv/ftp/client1` et
`client2` dans `/srv/ftp/client2`.

## 6. Creer les repertoires FTP

```bash
sudo install -d -o root -g root -m 755 /srv/ftp/client1
sudo install -d -o root -g root -m 755 /srv/ftp/client2
```

Pour permettre l'ecriture tout en gardant une racine de chroot non modifiable,
il est preferable de creer un sous-repertoire :

```bash
sudo chown root:root /srv/ftp/client1 /srv/ftp/client2
sudo install -d -o ftpvirtual -g ftpvirtual -m 755 /srv/ftp/client1/files
sudo install -d -o ftpvirtual -g ftpvirtual -m 755 /srv/ftp/client2/files
```

La racine reste en `root:root` et seul le sous-dossier `files` est inscriptible par `ftpvirtual`.

## 7. Redemarrer et verifier

```bash
sudo systemctl restart vsftpd
sudo systemctl status --no-pager vsftpd
sudo ss -ltnp | grep ':21 '
```

Si un pare-feu est actif, autoriser TCP `21` et la plage passive
`40000:40100`.

## 8. Tester une connexion

```bash
ftp 127.0.0.1
```

Utiliser `client1` et son mot de passe, puis tester :

```text
pwd
ls
cd files
put fichier-test.txt
```

Une verification non interactive est aussi possible :

```bash
curl --user 'client1:mot_de_passe' ftp://127.0.0.1/
```

Attention : placer un mot de passe directement dans une commande peut
l'enregistrer dans l'historique du shell.

## 9. Ajouter un utilisateur

Les noms sont limites aux lettres, chiffres, points, tirets et underscores.
Executer le bloc complet :

```bash
USER_NAME=nouveau_client
if ! printf "%s" "$USER_NAME" | grep -Eq "^[A-Za-z0-9._-]+$"; then
  echo "Nom utilisateur invalide" >&2
elif sudo grep -q "^${USER_NAME}:" /etc/vsftpd/virtual_users; then
  echo "Erreur : ${USER_NAME} existe deja" >&2
else
  read -rsp "Mot de passe FTP : " FTP_PASSWORD; echo
  HASH=$(printf "%s" "$FTP_PASSWORD" | openssl passwd -6 -stdin)
  unset FTP_PASSWORD
  printf "%s:%s\n" "$USER_NAME" "$HASH" | sudo tee -a /etc/vsftpd/virtual_users >/dev/null
  unset HASH
  sudo install -d -o root -g root -m 755 "/srv/ftp/$USER_NAME"
  sudo install -d -o ftpvirtual -g ftpvirtual -m 755 "/srv/ftp/$USER_NAME/files"
fi
unset USER_NAME
```

Le fichier est relu a chaque authentification : aucun redemarrage n’est normalement necessaire.

## 10. Changer un mot de passe

```bash
USER_NAME=client1
if ! sudo grep -q "^${USER_NAME}:" /etc/vsftpd/virtual_users; then
  echo "Erreur : ${USER_NAME} n’existe pas" >&2
else
  read -rsp "Nouveau mot de passe FTP : " FTP_PASSWORD; echo
  HASH=$(printf "%s" "$FTP_PASSWORD" | openssl passwd -6 -stdin)
  unset FTP_PASSWORD
  sudo sed -i "s|^${USER_NAME}:.*|${USER_NAME}:$HASH|" /etc/vsftpd/virtual_users
  unset HASH
fi
unset USER_NAME
```

## 11. Supprimer un utilisateur

Sauvegarder puis retirer uniquement sa ligne :

```bash
sudo cp -a /etc/vsftpd/virtual_users /etc/vsftpd/virtual_users.backup
sudo sed -i '/^client1:/d' /etc/vsftpd/virtual_users
```

Archiver ou supprimer ensuite `/srv/ftp/client1` selon la politique de
conservation des donnees. Si ce chemin est un montage bind, le demonter avant
toute suppression :

```bash
mountpoint -q /srv/ftp/client1/files && sudo umount /srv/ftp/client1/files
```

## 12. Verifier qu'un compte n'existe pas dans Linux

```bash
id client1
```

Resultat attendu :

```text
id: 'client1': no such user
```

## 13. Donner acces a un dossier web

Exemple pour `/var/www/html/client1` :

```bash
sudo install -d -o ftpvirtual -g www-data -m 2775 /var/www/html/client1
sudo install -d -o root -g root -m 755 /srv/ftp/client1
sudo install -d -o ftpvirtual -g ftpvirtual -m 755 /srv/ftp/client1/files
sudo mount --bind /var/www/html/client1 /srv/ftp/client1/files
```

Pour rendre le montage permanent, ajouter dans `/etc/fstab` :

```fstab
/var/www/html/client1 /srv/ftp/client1/files none bind 0 0
```

Puis verifier :

```bash
sudo mount -a
findmnt /srv/ftp/client1/files
```

## 14. Diagnostic

Suivre les authentifications :

```bash
sudo journalctl -f | grep -Ei --line-buffered 'vsftpd|pam_pwdfile'
```

Verifier la configuration utile :

```bash
sudo grep -vE '^#|^$' /etc/vsftpd.conf
cat /etc/pam.d/vsftpd-virtual
sudo cut -d: -f1 /etc/vsftpd/virtual_users
id ftpvirtual
namei -l /srv/ftp/client1
```

Erreurs frequentes :

- `530 Login incorrect` et `wrong password` : hash ou mot de passe incorrect.
- `530 Login incorrect` et `user unknown` : compte absent du fichier virtuel.
- `PAM unable to dlopen(pam_pwdfile.so)` : installer `libpam-pwdfile`.
- `500 OOPS: cannot change directory` : repertoire absent ou droits incorrects.
- `500 OOPS: refusing to run with writable root inside chroot` : racine de chroot inscriptible ; utiliser un sous-dossier `files`.
- Connexion possible mais `ls` bloque : plage passive fermee par le pare-feu.
- `id client1` echoue : comportement normal pour un utilisateur virtuel.


## Specifications validees

```text
Systeme                 Debian 13
vsftpd                  3.0.5
libpam-pwdfile          2.0-1+b1
OpenSSL                 3.5.6
Authentification        PAM avec fichier passwd
Hash recommande         SHA-512 crypt ($6$)
Compte Linux technique  ftpvirtual
Port de controle        TCP 21
Ports passifs           TCP 40000 a 40100
Racine FTP type         /srv/ftp/<utilisateur>
```

Le hash SHA-512 (`$6$`) a ete valide par une connexion FTP reelle. Le format Unix MD5 (`$1$`) fonctionne aussi mais est moins robuste. Le format Apache APR1 (`$apr1$`) genere par `htpasswd -m` a ete rejete sur cette installation.

Derriere un NAT, ajouter `pasv_address=ADRESSE_IP_PUBLIQUE` dans `vsftpd.conf` et ouvrir TCP 21 ainsi que TCP 40000 a 40100 sur le pare-feu et le routeur.

## Modifications appliquees sur ce serveur

- Installation de `libpam-pwdfile`.
- Activation de `pam_pwdfile.so` pour les utilisateurs virtuels.
- Protection de `/etc/vsftpd/virtual_users` en `root:root` avec le mode `600`.
- Utilisation du compte Linux technique `ftpvirtual`.
- Correction du hash du compte virtuel `yuto` vers un format Unix compatible.
- Configuration de la racine de `yuto` sur `/srv/ftp-jail/vboxuser`.
- Redemarrage de `vsftpd` et validation par une connexion FTP locale.

La configuration active de cette machine utilise `pam_service_name=vsftpd`, donc `/etc/pam.d/vsftpd`. La configuration generique du guide utilise `vsftpd-virtual` afin de separer clairement le profil des comptes virtuels.
