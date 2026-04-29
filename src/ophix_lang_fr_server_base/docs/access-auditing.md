---
title: Journal d'audit d'accès
slug: access-auditing
order: 30
section: Prise en main
---

Chaque serveur Ophix enregistre un journal des événements d'accès aux artefacts des clients. Le journal d'audit est intégré à `ophix-server-base` et ne nécessite aucune configuration supplémentaire — il est actif sur tous les serveurs de domaine (identifiants, configurations, certificats, et tout futur domaine).

---

## Ce qui est journalisé

Chaque fois qu'un client lit ou écrit un artefact avec succès, un enregistrement `AccessLog` est créé avec :

| Champ | Description |
| --- | --- |
| `client` | Le client ayant effectué la requête |
| `host` | L'hôte sur lequel le client s'exécute |
| `operation` | `GET` (lecture) ou `POST` (écriture/création) |
| `artifact_type` | Nom de la classe du modèle — `Credential`, `CertBundle`, etc. |
| `artifact_name` | Nom de l'artefact au moment de l'accès (instantané — survivant aux renommages et suppressions) |
| `artifact_id` | Clé primaire de l'artefact au moment de l'accès (survit à la suppression de l'artefact) |
| `timestamp` | Heure de la requête (définie à la création de l'événement, non à l'écriture en base de données) |

Seules les opérations réussies sont journalisées. Les requêtes rejetées par les contrôles d'authentification ou de permission n'apparaissent pas dans le journal d'audit (elles sont enregistrées dans le journal applicatif via la journalisation standard de Django).

---

## Consultation du journal d'audit

Le journal d'audit est accessible dans l'administration Django sous **Ophix Core → Journaux d'accès**.

La vue liste supporte le filtrage par :

- Client
- Hôte
- Opération (GET / POST)
- Type d'artefact
- Date (navigation par hiérarchie de dates)

Le journal est en lecture seule — les enregistrements ne peuvent pas être ajoutés, modifiés ou supprimés via l'administration.

---

## Performance

L'enregistreur d'audit est non bloquant par conception. Les vues de domaine placent les événements dans une file en mémoire et retournent immédiatement — le thread de requête n'attend jamais une écriture en base de données. Un thread démon en arrière-plan vide la file et écrit des lots via un seul appel `bulk_create()`.

Si la file se remplit sous une charge extrême, les événements excédentaires sont ignorés et un avertissement est écrit dans le journal applicatif. L'audit n'introduit jamais de contre-pression sur le service de requêtes principal.

---

## Paramètres

| Variable | Défaut | Description |
| --- | --- | --- |
| `AUDIT_BATCH_SIZE` | `50` | Nombre d'événements à accumuler avant d'écrire un lot en base de données. |
| `AUDIT_FLUSH_INTERVAL` | `5` | Délai maximal en secondes avant de vider un lot partiel. Garantit que les événements ne sont pas retenus indéfiniment lors d'un faible trafic. |

Ces paramètres peuvent être définis dans `.env`. Les valeurs par défaut conviennent à la plupart des déploiements.

---

## Élagage des anciens enregistrements

Les enregistrements s'accumulent indéfiniment sans élagage. Utilisez la commande de gestion `prune_access_log` pour supprimer les anciens enregistrements :

```bash
# Supprimer les enregistrements de plus de 90 jours (par défaut)
ophix-manage prune_access_log

# Supprimer les enregistrements de plus de 30 jours
ophix-manage prune_access_log --days 30

# Aperçu du nombre d'enregistrements qui seraient supprimés
ophix-manage prune_access_log --dry-run
ophix-manage prune_access_log --days 30 --dry-run
```

Exécutez cette commande selon un calendrier — une tâche cron quotidienne ou hebdomadaire est typique :

```cron
0 3 * * 0   ophixuser  /path/to/venv/bin/ophix-manage prune_access_log --days 90
```

Exécutez toujours `--dry-run` en premier sur un serveur de production avant d'ajuster la période de rétention, afin de confirmer le nombre d'enregistrements qui seront supprimés.
