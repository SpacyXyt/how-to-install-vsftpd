# 03 — Choisir le mode de gestion des utilisateurs

Avant de configurer les utilisateurs FTP, il faut choisir une méthode.

`vsftpd` peut fonctionner avec deux approches principales :

```txt
Mode A : utilisateurs Linux classiques
Mode B : utilisateurs virtuels FTP
````

---

# Mode A — Utilisateurs Linux

## Principe

Chaque utilisateur FTP est aussi un utilisateur Linux.

Exemple :

```bash
sudo adduser ftpuser
```

L’utilisateur existe alors dans le système :

```bash
getent passwd ftpuser
```

Exemple de résultat :

```txt
ftpuser:x:1001:1001::/home/ftpuser:/bin/bash
```

## Avantages

* Simple à configurer
* Facile à comprendre
* Compatible directement avec les permissions Linux
* Pratique pour une petite machine ou une VM de test

## Inconvénients

* Les utilisateurs FTP existent vraiment côté Linux
* Mauvais choix si tu veux créer beaucoup de comptes clients
* Il faut bloquer le shell si tu ne veux pas qu’ils puissent se connecter autrement

## Sécurisation recommandée

Pour empêcher l’utilisateur d’avoir un shell :

```bash
sudo usermod -s /usr/sbin/nologin ftpuser
```

Vérifier que `/usr/sbin/nologin` est autorisé comme shell :

```bash
cat /etc/shells
```

Si la ligne n’existe pas, l’ajouter :

```bash
echo "/usr/sbin/nologin" | sudo tee -a /etc/shells
```

## Documentation du mode A

Voir :

```txt
04A-utilisateurs-linux.md
```

---

# Mode B — Utilisateurs virtuels FTP

## Principe

Les utilisateurs FTP sont stockés dans un fichier dédié.

Ils n’existent pas comme utilisateurs Linux.

Exemple :

```txt
/etc/vsftpd/virtual_users
```

Un utilisateur virtuel comme `client1` peut se connecter en FTP, mais il n’existe pas dans Linux :

```bash
id client1
```

Résultat attendu :

```txt
id: ‘client1’: no such user
```

Tous les utilisateurs virtuels sont mappés sur un seul utilisateur Linux technique.

Exemple :

```txt
ftpvirtual
```

## Avantages

* Séparation propre entre Linux et FTP
* Plus sécurisé pour un système d’hébergement
* Les clients FTP n’ont pas de compte Linux
* Plus simple à automatiser dans un panel ou une API
* Chaque utilisateur peut avoir son propre dossier

## Inconvénients

* Configuration plus longue
* Nécessite PAM
* Les permissions fichiers passent par un utilisateur Linux technique commun

## Documentation du mode B

Voir :

```txt
04B-utilisateurs-virtuels.md
```

---

# Quel mode choisir ?

## Pour une VM simple

Choisir :

```txt
Mode A — Utilisateurs Linux
```

## Pour un serveur web personnel

Choisir :

```txt
Mode A ou B
```

## Pour un système d’hébergement

Choisir :

```txt
Mode B — Utilisateurs virtuels FTP
```

## Pour créer des comptes clients

Choisir :

```txt
Mode B — Utilisateurs virtuels FTP
```

---

# Résumé

| Besoin                                 | Mode conseillé |
| -------------------------------------- | -------------- |
| Test rapide                            | Mode A         |
| Petite VM locale                       | Mode A         |
| Serveur web simple                     | Mode A ou B    |
| Hébergement multi-clients              | Mode B         |
| Séparer Linux et FTP                   | Mode B         |
| Automatiser la création de comptes FTP | Mode B         |

