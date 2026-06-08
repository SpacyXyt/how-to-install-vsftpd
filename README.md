# Documentation vsftpd

Cette documentation explique comment installer, configurer et sécuriser un serveur FTP avec `vsftpd` sur Debian.

## Structure

```txt
docs/vsftpd/
├── README.md
├── 01-installation.md
├── 02-configuration-obligatoire.md
├── 03-choix-mode-utilisateurs.md
├── 04A-utilisateurs-linux.md
├── 04B-utilisateurs-virtuels.md
├── 05-dossier-par-utilisateur.md
├── 06-sous-dossiers-autorises.md
├── 07-firewall.md
├── 08-tests-et-debug.md
└── 09-configurations-finales.md
````

## Étapes obligatoires

1. Installer `vsftpd`
   Voir : `01-installation.md`

2. Appliquer la configuration de base
   Voir : `02-configuration-obligatoire.md`

3. Choisir le mode de gestion des utilisateurs
   Voir : `03-choix-mode-utilisateurs.md`

## Deux modes possibles

### Mode A — Utilisateurs Linux

Dans ce mode, les comptes FTP sont de vrais utilisateurs Linux.

Exemple :

```txt
ftpuser existe dans /etc/passwd
ftpuser peut avoir un dossier /home/ftpuser
ftpuser peut être bloqué sans accès SSH
```

Documentation :

```txt
04A-utilisateurs-linux.md
```

Ce mode est plus simple à configurer.

### Mode B — Utilisateurs virtuels FTP

Dans ce mode, les comptes FTP n’existent pas comme utilisateurs Linux.

Exemple :

```txt
client1 existe seulement dans /etc/vsftpd/virtual_users
client1 n’existe pas dans /etc/passwd
client1 ne peut pas se connecter en SSH
```

Documentation :

```txt
04B-utilisateurs-virtuels.md
```

Ce mode est recommandé pour un système d’hébergement propre avec plusieurs clients.

## Étapes optionnelles

* Dossier différent par utilisateur
  Voir : `05-dossier-par-utilisateur.md`

* Autoriser uniquement certains sous-dossiers
  Voir : `06-sous-dossiers-autorises.md`

* Configuration firewall
  Voir : `07-firewall.md`

* Tests et debug
  Voir : `08-tests-et-debug.md`

* Configurations finales prêtes à copier
  Voir : `09-configurations-finales.md`

## Recommandation

Pour un usage simple :

```txt
Mode A — Utilisateurs Linux
```

Pour un système type hébergement web, panel, clients ou espaces séparés :

```txt
Mode B — Utilisateurs virtuels FTP
```

## Attention

Ne mélange pas les deux modes dans la même configuration sans savoir exactement ce que tu fais.

Choisis soit :

```txt
Mode A : utilisateurs Linux
```

soit :

```txt
Mode B : utilisateurs virtuels
```
