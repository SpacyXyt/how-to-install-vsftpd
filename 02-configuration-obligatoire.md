# 02 — Configuration obligatoire de vsftpd

## 1. Ouvrir la configuration

```bash
sudo nano /etc/vsftpd.conf
````

## 2. Configuration minimale recommandée

Remplacer ou adapter le fichier avec cette configuration :

```conf
listen=NO
listen_ipv6=YES

anonymous_enable=NO
local_enable=YES
write_enable=YES

local_umask=022

chroot_local_user=YES
allow_writeable_chroot=YES

userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO

pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

xferlog_enable=YES
log_ftp_protocol=YES
```

## 3. Explication des options importantes

### Désactiver les connexions anonymes

```conf
anonymous_enable=NO
```

Empêche les utilisateurs anonymes de se connecter.

### Autoriser les utilisateurs locaux

```conf
local_enable=YES
```

Permet aux utilisateurs Linux locaux de se connecter en FTP.

### Autoriser l’écriture

```conf
write_enable=YES
```

Permet l’upload, la modification et la suppression de fichiers selon les permissions Linux.

### Bloquer les utilisateurs dans leur dossier

```conf
chroot_local_user=YES
```

Empêche les utilisateurs FTP de sortir de leur dossier racine FTP.

### Autoriser uniquement les utilisateurs listés

```conf
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
```

Avec `userlist_deny=NO`, seuls les utilisateurs présents dans `/etc/vsftpd.userlist` peuvent se connecter.

### Activer le mode passif

```conf
pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100
```

Le mode passif est nécessaire pour éviter beaucoup de problèmes de connexion FTP derrière NAT ou firewall.

## 4. Redémarrer vsftpd

```bash
sudo systemctl restart vsftpd
```

## 5. Vérifier le service

```bash
sudo systemctl status vsftpd
```
