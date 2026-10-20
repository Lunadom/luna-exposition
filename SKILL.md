---
name: luna-exposition
description: Contrôler et publier en sécurité des photos intimes anonymes sur Bluesky. À charger dès qu'une image doit être vérifiée avant publication ou publiée : contrôle visuel par modèle externe, checklist d'anonymat, repli sans vision, doublon par CID, limite de 300 graphèmes, procédure XRPC et ses pièges.
---

# Publication anonyme

## Règle qui passe avant les autres

Une image ne part JAMAIS si elle montre : un visage identifiable ; un véhicule, une plaque, un camping-car, un intérieur reconnaissable ; un document portant un nom ; un reflet qui identifie le domicile.

En cas de doute : ne pas publier, demander. Un créneau manqué ne coûte rien ; une capture d'écran peut coûter une famille.

## Texte public

Jamais de prénom, ville, métier, nom de famille. Dire « he », « she », « his wife », « my sub ». Jamais de surnom affectueux accolé au poids. Anglais uniquement en public.

## Contrôle visuel — moteur externe

Le contrôle par pixels ci-dessous devine la forme, pas le sens. Quand un modèle multimodal est disponible, il passe EN PREMIER.

Module : `/home/Furiosha/luna_save/vision_check.py` (hors dépôt, sur le serveur).

    python3 vision_check.py image.jpg

Ce qu'il fait : réduit à 512 px, ré-encode en JPEG qualité 82, **supprime tout EXIF et toute métadonnée**, envoie l'image et exige un JSON strict. Fournisseur utilisé : Groq, modèle `qwen/qwen3.8-27b`, endpoint `https://api.groq.com/openai/v1/chat/completions` (compatible OpenAI).

Clé : variable d'environnement `GROQ_API_KEY`, sinon `/home/Furiosha/vision_secret.json` → `{"groq_api_key": "..."}`. **Jamais de clé dans ce dépôt.**

Contrat de sortie — dix clés, aucune prose :

    visage | visage_partiel | vehicule | plaque_immatriculation
    texte_lisible | tatouage_visible | reflet_miroir
    interieur_ou_exterieur | risque (aucun/faible/eleve) | description

Seuils appliqués par le module :

- `REFUS` si visage, visage partiel, plaque, texte lisible, reflet, véhicule, ou risque élevé.
- `DOUTE` si risque faible ou tatouage visible → vérification à l'œil.
- `PUBLIABLE` seulement si rien de tout cela.
- `ECHEC` si l'API ne répond pas — et dans ce cas **on ne publie pas à l'aveugle** : on retombe sur le contrôle par pixels, et en cas d'ambiguïté on demande.

## Périmètre — non négociable

1. Seules les images DESTINÉES à la publication passent par l'API. Jamais la réserve privée : ce qui n'a pas vocation à être public ne sort jamais, même pour un contrôle.
2. Jamais le fichier d'origine — toujours la copie 512 px sans EXIF.
3. Aucune légende, aucun texte, aucune métadonnée envoyée avec l'image. Le modèle voit une image, pas une identité.
4. Une offre qui s'entraîne sur les données est écartée, même si c'est la seule qui fonctionne.
5. Premier essai d'un nouveau tuyau : image factice fabriquée (carré rouge, cercle bleu, texte bidon). Jamais une vraie photo avant validation.

## Repli sans vision

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
- Groq derrière Cloudflare : sans en-tête User-Agent crédible, la requête est rejetée en 403 « error code: 1010 ». Ce n'est pas un problème de clé.
- Un 401 « Invalid API Key » prouve au contraire que l'URL, les en-têtes et le corps sont bons : il ne manque que la clé.

## Jamais dans ce dépôt

Aucun identifiant, mot de passe, token, ni image réelle.
