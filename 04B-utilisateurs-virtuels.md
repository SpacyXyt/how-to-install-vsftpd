# 04B — Utilisateurs virtuels vsftpd

Cette méthode permet de créer des utilisateurs FTP qui ne sont pas des utilisateurs Linux.

## Objectif

Avoir des comptes FTP séparés du système Linux.

Exemple :

```txt
client1 = utilisateur FTP uniquement
client2 = utilisateur FTP uniquement
ftpvirtual = utilisateur Linux technique
````

Les utilisateurs `client1` et `client2` n’existent pas dans Linux.

Ils sont seulement utilisés par `vsftpd`.

---

# 1. Installer les paquets nécessaires

```bash
sudo apt update
sudo apt install vsftpd apache2-utils libpam-pwdfile -y
```

Paquets utilisés :

```txt
vsftpd          serveur FTP
apache2-utils   outil htpasswd
libpam-pwdfile  authentification PAM via fichier de mots de passe
```

---

# 2. Créer l’utilisateur Linux technique

Cet utilisateur ne sert pas à se connecter directement.

Il sert uniquement à donner une identité Linux aux utilisateurs virtuels.

```bash
sudo useradd -d /srv/ftp -s /usr/sbin/nologin ftpvirtual
```

Créer le dossier principal :

```bash
sudo mkdir -p /srv/ftp
sudo chown root:root /srv/ftp
sudo chmod 755 /srv/ftp
```

Vérifier l’utilisateur technique :

```bash
id ftpvirtual
```

---

# 3. Créer le fichier des utilisateurs virtuels

Créer le dossier de configuration :

```bash
sudo mkdir -p /etc/vsftpd
```

Créer le premier utilisateur virtuel :

```bash
sudo htpasswd -c /etc/vsftpd/virtual_users client1
```

Ajouter un autre utilisateur virtuel :

```bash
sudo htpasswd /etc/vsftpd/virtual_users client2
```

Ne pas remettre `-c` pour les utilisateurs suivants, sinon le fichier sera écrasé.

Sécuriser le fichier :

```bash
sudo chown root:root /etc/vsftpd/virtual_users
sudo chmod 600 /etc/vsftpd/virtual_users
```

Voir les utilisateurs virtuels :

```bash
sudo cat /etc/vsftpd/virtual_users
```

---

# 4. Créer la configuration PAM

Créer un fichier PAM dédié :

```bash
sudo nano /etc/pam.d/vsftpd-virtual
```

Mettre :

```pam
auth required pam_pwdfile.so pwdfile /etc/vsftpd/virtual_users
account required pam_permit.so
```

---

# 5. Sauvegarder la configuration vsftpd actuelle

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
```

---

# 6. Configurer vsftpd pour les utilisateurs virtuels

Ouvrir la configuration :

```bash
sudo nano /etc/vsftpd.conf
```

Configuration recommandée :

```conf
listen=YES
listen_ipv6=NO

anonymous_enable=NO
local_enable=YES
write_enable=YES

guest_enable=YES
guest_username=ftpvirtual
virtual_use_local_privs=YES

pam_service_name=vsftpd-virtual

chroot_local_user=YES
allow_writeable_chroot=YES

user_sub_token=$USER
local_root=/srv/ftp/$USER

pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

xferlog_enable=YES
log_ftp_protocol=YES
```

---

# 7. Explication des options importantes

## Activer les utilisateurs virtuels

```conf
guest_enable=YES
guest_username=ftpvirtual
```

Les utilisateurs virtuels sont mappés sur l’utilisateur Linux technique `ftpvirtual`.

## Utiliser PAM

```conf
pam_service_name=vsftpd-virtual
```

`vsftpd` utilise le fichier :

```txt
/etc/pam.d/vsftpd-virtual
```

## Créer un dossier différent par utilisateur virtuel

```conf
user_sub_token=$USER
local_root=/srv/ftp/$USER
```

Exemple :

```txt
client1 arrive dans /srv/ftp/client1
client2 arrive dans /srv/ftp/client2
```

---

# 8. Créer les dossiers des utilisateurs virtuels

Pour `client1` :

```bash
sudo mkdir -p /srv/ftp/client1
sudo chown -R ftpvirtual:ftpvirtual /srv/ftp/client1
sudo chmod 755 /srv/ftp/client1
```

Pour `client2` :

```bash
sudo mkdir -p /srv/ftp/client2
sudo chown -R ftpvirtual:ftpvirtual /srv/ftp/client2
sudo chmod 755 /srv/ftp/client2
```

---

# 9. Redémarrer vsftpd

```bash
sudo systemctl restart vsftpd
```

Vérifier le statut :

```bash
sudo systemctl status vsftpd
```

---

# 10. Tester la connexion

Depuis le serveur :

```bash
ftp 127.0.0.1
```

Identifiants :

```txt
Utilisateur : client1
Mot de passe : celui défini avec htpasswd
```

Tester :

```ftp
pwd
ls
cd ..
ls
```

L’utilisateur doit rester bloqué dans son dossier FTP.

---

# 11. Ajouter un utilisateur virtuel plus tard

Ajouter l’utilisateur :

```bash
sudo htpasswd /etc/vsftpd/virtual_users nouveau_client
```

Créer son dossier :

```bash
sudo mkdir -p /srv/ftp/nouveau_client
sudo chown -R ftpvirtual:ftpvirtual /srv/ftp/nouveau_client
sudo chmod 755 /srv/ftp/nouveau_client
```

Redémarrer :

```bash
sudo systemctl restart vsftpd
```

---

# 12. Supprimer un utilisateur virtuel

Supprimer l’utilisateur du fichier :

```bash
sudo htpasswd -D /etc/vsftpd/virtual_users client1
```

Supprimer ou archiver son dossier :

```bash
sudo rm -rf /srv/ftp/client1
```

Redémarrer :

```bash
sudo systemctl restart vsftpd
```

---

# 13. Changer le mot de passe d’un utilisateur virtuel

```bash
sudo htpasswd /etc/vsftpd/virtual_users client1
```

---

# 14. Vérifier qu’un utilisateur virtuel n’existe pas dans Linux

```bash
id client1
```

Résultat attendu :

```txt
id: ‘client1’: no such user
```

Cela confirme que `client1` est uniquement un utilisateur FTP.

---

# 15. Donner accès à un dossier web

Exemple : `client1` doit gérer un site web situé dans :

```txt
/var/www/html/client1
```

Créer le dossier :

```bash
sudo mkdir -p /var/www/html/client1
```

Donner les droits à l’utilisateur technique :

```bash
sudo chown -R ftpvirtual:www-data /var/www/html/client1
sudo chmod -R 775 /var/www/html/client1
```

Remplacer le dossier FTP de `client1` par un lien de montage :

```bash
sudo rm -rf /srv/ftp/client1
sudo mkdir -p /srv/ftp/client1
sudo mount --bind /var/www/html/client1 /srv/ftp/client1
```

Rendre le montage permanent :

```bash
sudo nano /etc/fstab
```

Ajouter :

```fstab
/var/www/html/client1 /srv/ftp/client1 none bind 0 0
```

Tester :

```bash
sudo mount -a
```

---

# 16. Debug

Voir les logs :

```bash
sudo journalctl -u vsftpd -f
```

Voir la configuration active :

```bash
sudo grep -vE '^#|^$' /etc/vsftpd.conf
```

Vérifier PAM :

```bash
cat /etc/pam.d/vsftpd-virtual
```

Vérifier le fichier des utilisateurs virtuels :

```bash
sudo cat /etc/vsftpd/virtual_users
```

Vérifier les ports :

```bash
sudo ss -tulpn | grep :21
```

---

# 17. Configuration finale mode utilisateurs virtuels

Fichier :

```txt
/etc/vsftpd.conf
```

Contenu :

```conf
listen=YES
listen_ipv6=NO

anonymous_enable=NO
local_enable=YES
write_enable=YES

guest_enable=YES
guest_username=ftpvirtual
virtual_use_local_privs=YES

pam_service_name=vsftpd-virtual

chroot_local_user=YES
allow_writeable_chroot=YES

user_sub_token=$USER
local_root=/srv/ftp/$USER

pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

xferlog_enable=YES
log_ftp_protocol=YES
```
