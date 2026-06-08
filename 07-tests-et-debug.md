# 07 — Tests et debug de vsftpd

## 1. Tester la connexion FTP en local

```bash
ftp 127.0.0.1
````

## 2. Tester depuis une autre machine

```bash
ftp IP_DU_SERVEUR
```

Exemple :

```bash
ftp 10.30.20.16
```

## 3. Commandes FTP utiles

```ftp
pwd
ls
cd ..
ls
put fichier.txt
get fichier.txt
quit
```

## 4. Vérifier le statut du service

```bash
sudo systemctl status vsftpd
```

## 5. Voir les logs systemd

```bash
sudo journalctl -u vsftpd -f
```

## 6. Voir les logs FTP

```bash
sudo tail -f /var/log/vsftpd.log
```

## 7. Voir la configuration active sans commentaires

```bash
sudo grep -vE '^#|^$' /etc/vsftpd.conf
```

## 8. Vérifier les utilisateurs autorisés

```bash
cat /etc/vsftpd.userlist
```

## 9. Vérifier le dossier de départ d’un utilisateur

```bash
getent passwd ftpuser
```

Exemple :

```txt
ftpuser:x:1001:1001::/home/ftpuser:/bin/bash
```

## 10. Tester les permissions comme l’utilisateur

```bash
sudo -u ftpuser ls -la /var/www/html
```

Tester un dossier précis :

```bash
sudo -u ftpuser ls -la /var/www/html/site1
```

## 11. Erreur : Login incorrect

Vérifier que l’utilisateur est dans :

```bash
cat /etc/vsftpd.userlist
```

Vérifier aussi que l’utilisateur existe :

```bash
id ftpuser
```

## 12. Erreur : Permission denied

Vérifier les droits du dossier :

```bash
ls -la /chemin/du/dossier
```

Donner les droits si nécessaire :

```bash
sudo setfacl -R -m u:ftpuser:rwx /chemin/du/dossier
```

## 13. Erreur : Cannot change directory

Vérifier le dossier racine configuré :

```bash
cat /etc/vsftpd_user_conf/ftpuser
```

Vérifier que le dossier existe :

```bash
ls -ld /srv/ftp/ftpuser
```

## 14. Redémarrer vsftpd

```bash
sudo systemctl restart vsftpd
```
