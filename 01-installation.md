# 01 — Installation de vsftpd

## 1. Mettre à jour le système

```bash
sudo apt update
sudo apt upgrade -y
````

## 2. Installer vsftpd

```bash
sudo apt install vsftpd -y
```

## 3. Activer et démarrer vsftpd

```bash
sudo systemctl enable --now vsftpd
```

## 4. Vérifier le statut

```bash
sudo systemctl status vsftpd
```

Le service doit être actif.

Exemple attendu :

```txt
Active: active (running)
```

## 5. Sauvegarder la configuration d’origine

```bash
sudo cp /etc/vsftpd.conf /etc/vsftpd.conf.backup
```
