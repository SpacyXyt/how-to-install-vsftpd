# 03 — Autoriser uniquement certains utilisateurs

## 1. Créer un utilisateur FTP

Exemple avec `ftpuser` :

```bash
sudo adduser ftpuser
````

Définir un mot de passe quand demandé.

## 2. Créer le fichier des utilisateurs autorisés

```bash
sudo nano /etc/vsftpd.userlist
```

Ajouter un utilisateur par ligne :

```txt
ftpuser
vboxuser
devlynx
```

## 3. Sécuriser le fichier

```bash
sudo chown root:root /etc/vsftpd.userlist
sudo chmod 600 /etc/vsftpd.userlist
```

## 4. Redémarrer vsftpd

```bash
sudo systemctl restart vsftpd
```

## 5. Ajouter un utilisateur à la liste plus tard

```bash
echo "nouvel_utilisateur" | sudo tee -a /etc/vsftpd.userlist
sudo systemctl restart vsftpd
```

## 6. Retirer un utilisateur de la liste

Ouvrir le fichier :

```bash
sudo nano /etc/vsftpd.userlist
```

Supprimer la ligne de l’utilisateur.

Puis redémarrer :

```bash
sudo systemctl restart vsftpd
```

## 7. Vérifier les utilisateurs autorisés

```bash
cat /etc/vsftpd.userlist
```

## 8. Attention

Ne pas mettre :

```conf
userlist_deny=YES
```

Sinon `/etc/vsftpd.userlist` devient une blacklist.

Pour autoriser uniquement les utilisateurs listés, il faut :

```conf
userlist_deny=NO
```
