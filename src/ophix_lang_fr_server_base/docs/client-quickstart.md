---
title: Démarrage rapide client
slug: client-quickstart
order: 20
section: Prise en main
---

Chaque client Ophix de niveau 1 suit le même processus d'initialisation. Les exemples ci-dessous utilisent `cred-client` — remplacez-le par `cert-client`, `conf-client`, etc. selon le domaine.

---

## Installation

```bash
pip install ophix-cred-client
```

---

## Initialisation en une seule étape

```bash
cred-client quickstart https://credserver.internal mon-client
```

Cette commande :

1. Enregistre l'URL du serveur dans `.cred.env`
2. Télécharge le certificat CA du serveur et l'enregistre localement
3. Enregistre ce client auprès du serveur et sauvegarde le jeton API

C'est tout — le client est maintenant prêt à être utilisé.

---

## Étape par étape

Si vous préférez contrôler chaque étape :

```bash
cred-client set server https://credserver.internal
cred-client download ca-cert
cred-client register mon-client
```

---

## Vérification

```bash
cred-client doctor
```

Affiche l'état de la configuration : URL du serveur, certificat CA, jeton et connectivité.

---

## Fichier de configuration

La configuration est stockée dans `.cred.env` (dans le répertoire courant ou dans `~`). Exemple :

```ini
CREDSERVER_URL=https://credserver.internal
CREDSERVER_CA_CERT=/home/user/.cred-ca.pem
CREDSERVER_API_TOKEN=<jeton hex 64 caractères>
```

---

## Rotation du jeton

Les jetons doivent être renouvelés régulièrement. La rotation génère un nouveau jeton, le valide auprès du serveur, puis met à jour le fichier de configuration :

```bash
cred-client rotate-token
```

Pour planifier la rotation (cron) :

```cron
0 2 * * 1   user  /path/to/venv/bin/cred-client rotate-token
```

La rotation échoue bruyamment si le serveur est inaccessible — ne la passez jamais en mode silencieux.

---

## Accorder l'accès aux artefacts

Les clients n'ont accès à aucun artefact par défaut. Un administrateur doit les lier explicitement depuis l'interface d'administration Django :

1. Ouvrir la page de détail du **Client** ou de l'**Artefact** dans l'administration
2. Utiliser l'inline pour créer le lien
3. Cocher **Enabled** sur le lien

Le client peut récupérer l'artefact dès que le lien est activé.
