# 06 — Firewall pour vsftpd

Cette étape est optionnelle, mais recommandée si un firewall est actif.

## 1. Ports utilisés

vsftpd utilise :

```txt
21 TCP              FTP
40000-40100 TCP     Ports passifs
````

## 2. Ouvrir les ports avec UFW

```bash
sudo ufw allow 21/tcp
sudo ufw allow 40000:40100/tcp
sudo ufw reload
```

## 3. Vérifier les règles UFW

```bash
sudo ufw status numbered
```

## 4. Vérifier que vsftpd écoute

```bash
sudo ss -tulpn | grep :21
```

Exemple attendu :

```txt
tcp LISTEN 0 32 0.0.0.0:21
```

## 5. Si le serveur est derrière une box ou un NAT

Il faut rediriger ces ports vers le serveur :

```txt
21 TCP
40000-40100 TCP
```

## 6. Option IP publique pour le mode passif

Si le serveur est accessible depuis Internet, ajouter dans `/etc/vsftpd.conf` :

```conf
pasv_address=IP_PUBLIQUE_DU_SERVEUR
```

Exemple :

```conf
pasv_address=156.67.30.4
```

Puis redémarrer :

```bash
sudo systemctl restart vsftpd
```
