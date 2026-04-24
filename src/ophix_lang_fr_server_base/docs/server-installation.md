---
title: Installation du serveur
slug: server-installation
order: 1
section: Prise en main
---

Ce guide couvre l'installation d'un serveur Ophix à partir de zéro. Chaque serveur est une instance Django autonome — un paquet de domaine pip installé au-dessus de `ophix-server-base`.

---

## Prérequis

- Python 3.10+
- MariaDB (moteur par défaut) ou un autre moteur supporté
- Un serveur web (nginx recommandé)
- Un accès root ou sudo pour l'installation du service

---

## Installation guidée

La séquence recommandée utilise deux commandes de gestion :

### Étape 1 — Configurer l'installation

```bash
ophix-manage configure_install credserver
```

L'assistant interactif collecte :

- Répertoire d'installation
- Nom d'hôte et certificats TLS (avec validation du CN/SAN)
- Identifiants de base de données (test de connexion en direct)
- Identifiants du superutilisateur
- Activation du thème et titre de l'administration

Le résultat est écrit dans `.<slug>.conf` et `.env` (chmod 600 les deux).

### Étape 2 — Exécuter l'installation

```bash
ophix-manage run_install credserver
```

Cette commande :

- Crée la structure de répertoires dans `INSTALL_DIR`
- Copie les fichiers TLS dans `ssl/certs/` et `ssl/private/`
- Génère les fichiers de configuration nginx et systemd
- Exécute `migrate`, `collectstatic --noinput`
- Crée ou met à jour le superutilisateur
- Active le thème et définit le titre de l'administration
- Importe automatiquement la documentation inline

### Étape 3 — Installer le service système

```bash
sudo bash credserver_sudo_install.sh
```

Configure les permissions, installe la configuration nginx, installe et démarre le service systemd.

---

## Mise à niveau

Pour une mise à niveau de routine :

```bash
pip install --upgrade ophix-server-base ophix-creds
ophix-manage migrate
ophix-manage collectstatic --noinput
sudo systemctl restart credserver
```

Si de nouvelles variables `.env` ont été ajoutées, exécutez d'abord `generate_deploy_config --append`.

---

## Configuration en développement

Pour un environnement de développement local :

```bash
python -m venv venv
source venv/bin/activate
pip install ophix-server-base ophix-creds ophix-docs
ophix-manage configure_install credserver
ophix-manage run_install credserver
ophix-manage runserver
```

---

## Référence des paramètres

### Identité du serveur

| Variable | Défaut | Description |
| --- | --- | --- |
| `SERVER_NAME` | _(requis)_ | Nom d'affichage de cette instance serveur |
| `SERVER_VERSION` | _(requis)_ | Version du logiciel serveur déployé |
| `INSTALL_DIR` | _(requis)_ | Répertoire racine de l'installation |
| `ALLOWED_HOSTS` | _(requis)_ | Noms d'hôtes autorisés (séparés par des virgules) |
| `DEBUG` | `False` | Mode debug Django. Ne jamais activer en production |

### Sécurité

| Variable | Défaut | Description |
| --- | --- | --- |
| `SECRET_KEY` | _(requis)_ | Clé secrète Django. Générée automatiquement lors de l'installation |
| `AUTH_LEAK_INFO` | `False` | Inclure les détails d'erreur dans les réponses API. `True` en développement uniquement |
| `MINIMUM_TOKEN_ROTATE_TIME` | `3600` | Délai minimum en secondes entre les rotations de jeton |

### Base de données

| Variable | Défaut | Description |
| --- | --- | --- |
| `DB_ENGINE` | `mariadb` | Moteur de base de données. Options : `mariadb`, `postgres`, `mssql`, `oracle`, `cockroachdb` |
| `DB_NAME` | _(requis)_ | Nom de la base de données |
| `DB_USER` | _(requis)_ | Utilisateur de la base de données |
| `DB_PASSWORD` | _(requis)_ | Mot de passe de la base de données |
| `DB_HOST` | `localhost` | Hôte de la base de données |
| `DB_PORT` | _(défaut moteur)_ | Port de la base de données |

### Localisation

| Variable | Défaut | Description |
| --- | --- | --- |
| `LANGUAGE_CODE` | `en-us` | Code de langue Django (ex. `fr`, `de`, `ru`) |
| `TIME_ZONE` | `UTC` | Fuseau horaire du serveur |

### Interface d'administration

| Variable | Défaut | Description |
| --- | --- | --- |
| `ADMIN_SITE_HEADER` | `Ophix` | Titre de l'en-tête de l'administration |
| `SHOW_CLIENT_ARTIFACT_MODEL` | `False` | Afficher le modèle de lien client-artefact dans l'administration |
| `SHOW_ACCESS_LOGS` | `False` | Afficher la section journaux d'accès dans l'administration |
| `SHOW_AUTH_MODELS` | `False` | Afficher les modèles d'authentification Django dans l'administration |
| `SHOW_THEME_MODEL` | `False` | Afficher le modèle de thème dans l'administration |

### Journal d'audit

| Variable | Défaut | Description |
| --- | --- | --- |
| `AUDIT_BATCH_SIZE` | `50` | Nombre d'événements par lot d'écriture |
| `AUDIT_FLUSH_INTERVAL` | `5` | Délai de vidage maximal en secondes |

### Notes de production

- Toujours déployer derrière un proxy inverse (nginx)
- Le proxy doit définir `X-Forwarded-For` de manière fiable — l'authentification Ophix dépend de l'IP source
- Garder `DEBUG=False` en production
- Sauvegarder le fichier `.env` séparément de la base de données — il contient les clés de chiffrement
