# 08 — Configuration finale complète de vsftpd

Fichier :

```bash
/etc/vsftpd.conf
````

Configuration recommandée :

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

user_config_dir=/etc/vsftpd_user_conf

pasv_enable=YES
pasv_min_port=40000
pasv_max_port=40100

xferlog_enable=YES
log_ftp_protocol=YES
```

## Fichier des utilisateurs autorisés

Fichier :

```bash
/etc/vsftpd.userlist
```

Exemple :

```txt
ftpuser
vboxuser
devlynx
```

Droits recommandés :

```bash
sudo chown root:root /etc/vsftpd.userlist
sudo chmod 600 /etc/vsftpd.userlist
```

## Exemple de configuration utilisateur

Fichier :

```bash
/etc/vsftpd_user_conf/ftpuser
```

Exemple simple :

```conf
local_root=/srv/ftp/ftpuser
write_enable=YES
```

Exemple avec jail de sous-dossiers :

```conf
local_root=/srv/ftp-jail/ftpuser
write_enable=YES
```

## Redémarrage final

```bash
sudo systemctl restart vsftpd
```

## Vérification finale

```bash
sudo systemctl status vsftpd
sudo ss -tulpn | grep :21
sudo grep -vE '^#|^$' /etc/vsftpd.conf
```
