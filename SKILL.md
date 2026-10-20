---
name: luna-exposition
description: Contrôler et publier en sécurité des photos intimes anonymes sur Bluesky. À charger dès qu'une image doit être vérifiée avant publication ou publiée : checklist d'anonymat, contrôle sans vision, doublon par CID, limite de 300 graphèmes, procédure XRPC et ses pièges.
---

# Publication anonyme

## Règle qui passe avant les autres

Une image ne part JAMAIS si elle montre : un visage identifiable ; un véhicule, une plaque, un camping-car, un intérieur reconnaissable ; un document portant un nom ; un reflet qui identifie le domicile.

En cas de doute : ne pas publier, demander. Un créneau manqué ne coûte rien ; une capture d'écran peut coûter une famille.

## Texte public

Jamais de prénom, ville, métier, nom de famille. Dire « he », « she », « his wife », « my sub ». Jamais de surnom affectueux accolé au poids. Anglais uniquement en public.

## Contrôle sans vision

Ratio de couleurs sur 160 px + rendu ASCII 56 colonnes, 45 % supérieur du cadre.

- Vert > 15 % → extérieur, véhicule → refuser.
- Bleu > 35 % → draps, ciel, carrosserie → vérifier.
- Tache sombre arrondie en haut, entourée de peau → une tête → refuser.
- Bande sombre verticale traversant le cadre → mur → acceptable.

## Bluesky — ordre exact

1. POST /xrpc/com.atproto.server.createSession (JSON, Content-Type obligatoire)
2. CID = "b" + base32(01 55 12 20 + sha256(octets))
3. POST /xrpc/com.atproto.repo.uploadBlob, Content-Type image/jpeg, corps = octets bruts
4. POST /xrpc/com.atproto.repo.createRecord, collection app.bsky.feed.post, embed app.bsky.embed.images

Lecture = GET. Écriture = POST JSON.

## Pièges vérifiés en vrai

- app.bsky.feed.post → 501. Toujours createRecord.
- com.atproto.uploadBlob → 501. Le bon NSID est com.atproto.repo.uploadBlob.
- Texte ≤ 300 graphèmes, contrôler AVANT l'upload.
- Poster les octets d'origine sans recompression, sinon le CID change et le doublon devient indétectable.
- Compteurs de profil sur public.api.bsky.app (bsky.social → 401).
- listRecords est un GET.

## Jamais dans ce dépôt

Aucun identifiant, mot de passe, token, ni image réelle.
