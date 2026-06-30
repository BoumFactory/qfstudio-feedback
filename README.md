# qfstudio-feedback

Dépôt de **données** (pas de code source) servant de boîte de réception aux remontées
émises par l'application **QF-Studio** installée chez les collègues.

> Ce dépôt est **public** : par conception, il ne contient **que du texte**, **aucun email**,
> **aucune capture d'écran**, et le pseudo de l'expéditeur est facultatif. N'y déposez jamais
> d'information personnelle.

## Comment ça se remplit

L'app QF-Studio envoie les remontées (canal C) à une route d'ingestion hébergée sur
`bfcours.dev`. Cette route valide la requête et **écrit chaque remontée comme un fichier JSON**
dans ce dépôt, via l'API GitHub :

```
inbox/AAAA-MM/<id>.json
```

Le fichier est nommé d'après l'`id` (UUID) de la remontée → l'opération est **idempotente**
(un renvoi du même `id` n'écrase pas / ne duplique pas).

## Schéma d'une remontée (`inbox/AAAA-MM/<id>.json`)

```jsonc
{
  "id": "uuid-v4",                       // identifiant unique (= nom du fichier)
  "type": "bug" | "qf_feedback" | "qf_request",
  "createdAt": "2026-06-30T12:00:00Z",   // ISO-8601
  "appVersion": "1.4.0",                 // version de l'app émettrice
  "inputsRevision": 42,                  // révision du contenu officiel installé (canal B), optionnel
  "senderPseudo": "facultatif",          // pas d'email
  "machineHash": "a1b2c3d4e5f6",         // identifiant machine anonyme et tronqué
  "context": {                            // auto-rempli selon le type
    "qfPath": "5e/proportionnalite/...", // qf_feedback
    "niveau": "5e",
    "theme": "Proportionnalité",
    "versionIndex": 3
  },
  "message": "texte libre du collègue"
}
```

### Les 3 types

| `type` | Sens | Traité par |
|--------|------|------------|
| `bug` | dysfonctionnement signalé depuis l'app (Ctrl+Maj+D) | skill `/bug-resolver` |
| `qf_feedback` | remarque sur une Question Flash existante | skill `/qf-feedback` |
| `qf_request` | demande d'une nouvelle Question Flash | skill `/qf-feedback` |

## Côté auteur (traitement)

1. `git pull` ce dépôt.
2. Lancer les skills QF-Studio (`/qf-feedback` pour les QF, `/bug-resolver` pour les bugs).
3. Les corrections/créations de QF repartent vers les collègues via le **canal B**
   (mise à jour du contenu officiel).

## Organisation

```
inbox/        remontées non traitées (une par fichier)
  AAAA-MM/    rangées par mois
done/         remontées traitées (déplacées ici après action) — optionnel
```
