# Fichier : `05-sous-dossiers-autorises.md`

```markdown
# 05 — Autoriser uniquement certains sous-dossiers à un utilisateur

Cette étape est optionnelle.

Objectif :

```txt
L’utilisateur FTP ne voit pas tout /var/www/html.
Il voit uniquement les dossiers autorisés.
````

Exemple :

```txt
/var/www/html/site1
/var/www/html/site2/uploads
```

L’utilisateur verra seulement :

```txt
/
├── site1
└── uploads
```

## 1. Créer une racine FTP isolée

Exemple avec l’utilisateur `ftpuser` :

```bash
sudo mkdir -p /srv/ftp-jail/ftpuser
sudo chown root:root /srv/ftp-jail/ftpuser
sudo chmod 755 /srv/ftp-jail/ftpuser
```

## 2. Créer les points de montage visibles

```bash
sudo mkdir -p /srv/ftp-jail/ftpuser/site1
sudo mkdir -p /srv/ftp-jail/ftpuser/uploads
```

## 3. Monter uniquement les dossiers autorisés

```bash
sudo mount --bind /var/www/html/site1 /srv/ftp-jail/ftpuser/site1
sudo mount --bind /var/www/html/site2/uploads /srv/ftp-jail/ftpuser/uploads
```

## 4. Rendre les montages permanents

Ouvrir `/etc/fstab` :

```bash
sudo nano /etc/fstab
```

Ajouter :

```fstab
/var/www/html/site1 /srv/ftp-jail/ftpuser/site1 none bind 0 0
/var/www/html/site2/uploads /srv/ftp-jail/ftpuser/uploads none bind 0 0
```

Tester :

```bash
sudo mount -a
```

## 5. Configurer vsftpd pour cet utilisateur

Créer ou modifier :

```bash
sudo nano /etc/vsftpd_user_conf/ftpuser
```

Ajouter :

```conf
local_root=/srv/ftp-jail/ftpuser
write_enable=YES
```

## 6. Donner les droits d’écriture si nécessaire

Méthode simple :

```bash
sudo chown -R ftpuser:www-data /var/www/html/site1
sudo chmod -R 775 /var/www/html/site1
```

Pour le dossier uploads :

```bash
sudo chown -R ftpuser:www-data /var/www/html/site2/uploads
sudo chmod -R 775 /var/www/html/site2/uploads
```

## 7. Méthode plus propre avec ACL

Installer ACL :

```bash
sudo apt install acl -y
```

Donner les droits à l’utilisateur :

```bash
sudo setfacl -R -m u:ftpuser:rwx /var/www/html/site1
sudo setfacl -R -m u:ftpuser:rwx /var/www/html/site2/uploads
```

Définir les droits par défaut pour les futurs fichiers :

```bash
sudo setfacl -R -d -m u:ftpuser:rwx /var/www/html/site1
sudo setfacl -R -d -m u:ftpuser:rwx /var/www/html/site2/uploads
```

## 8. Redémarrer vsftpd

```bash
sudo systemctl restart vsftpd
```

## 9. Tester

```bash
ftp IP_DU_SERVEUR
```

Dans la session FTP :

```ftp
pwd
ls
cd ..
ls
```

L’utilisateur doit voir uniquement les dossiers montés.
