# 04 — Dossier FTP différent par utilisateur

Cette étape est optionnelle.

Elle permet de définir un dossier racine spécifique pour chaque utilisateur FTP.

## 1. Activer les configurations par utilisateur

Ouvrir la configuration principale :

```bash
sudo nano /etc/vsftpd.conf
````

Ajouter :

```conf
user_config_dir=/etc/vsftpd_user_conf
```

## 2. Créer le dossier des configurations utilisateurs

```bash
sudo mkdir -p /etc/vsftpd_user_conf
```

## 3. Créer un dossier FTP pour l’utilisateur

Exemple avec `ftpuser` :

```bash
sudo mkdir -p /srv/ftp/ftpuser
sudo chown ftpuser:ftpuser /srv/ftp/ftpuser
sudo chmod 755 /srv/ftp/ftpuser
```

## 4. Créer la configuration spécifique de l’utilisateur

```bash
sudo nano /etc/vsftpd_user_conf/ftpuser
```

Ajouter :

```conf
local_root=/srv/ftp/ftpuser
write_enable=YES
```

## 5. Redémarrer vsftpd

```bash
sudo systemctl restart vsftpd
```

## 6. Tester

Se connecter en FTP :

```bash
ftp IP_DU_SERVEUR
```

Puis :

```ftp
pwd
ls
cd ..
ls
```

L’utilisateur ne doit pas pouvoir sortir de son dossier FTP.
