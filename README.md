# 📦 Documentation API — Gestion des Colis : Youssel Transport

---

## 📚 Table des matières

1. [Authentification](#1-authentification)
2. [Liste des gouvernorats](#2-liste-des-gouvernorats)
3. [Création d'un colis](#3-création-dun-colis)
4. [Modification d'un colis](#4-modification-dun-colis)
5. [Historique d'un colis](#5-historique-dun-colis)
6. [Exemples d'utilisation](#6-exemples-dutilisation)
7. [Codes d'erreur](#7-codes-derreur)
8. [Règles métier](#8-règles-métier)

---

## 1️⃣ Authentification

L'API utilise **Bearer Token** pour tous les endpoints sauf la **liste des gouvernorats**.

**Workflow d'authentification :**
1. Obtenir un token via `POST /api/login`
2. Utiliser le token dans le header :  
   `Authorization: Bearer {token}`
3. Le token est valable **24 heures**

---

### 🔑 Endpoint d’authentification

**URL :** `POST /api/login`

**Body :**
```json
{
  "email": "votre@email.com",
  "password": "votre_mot_de_passe"
}
```
**Réponse succès :**

```json
{
  "user_id": 777,
  "token": "678|IcH4mmRg9HLTSOdA3caFuEtK6YQenxbTHBV2Q5Wqc2bbb524"
}
```
### 2️⃣ Liste des gouvernorats
**⚠️ PUBLIC — Aucun token requis**

**URL :** `GET /api/gouvernorat`

**Réponse:**
```json
{
  "status": "success",
  "data": [
    {
      "id": 1,
      "name": "Tunis",
      "bigboss_name": "TUNIS"
    }
  ]
}
```
### 3️⃣ Création d’un colis
**🔒 PROTÉGÉ — Token requis**

**URL :** `POST /api/colis/v1/ajouter`

**Headers :**

```json
Authorization: Bearer {token}
Content-Type: application/json
```

**Body obligatoire**

Ce sont les champs **requis** pour la création standard d’un colis :


```json
{
  "nom_complet": "Nom du destinataire",
  "adresse": "Adresse de livraison",
  "tel1": "12345678",
  "nombre_pieces": 1,
  "service": "Livraison",
  "id_gov": 1
}
```

**Champs optionnels (non obligatoires)**

| Champ     | Type               | Description |
|------------|--------------------|--------------|
| `tel2`     | string (8 chiffres) | Numéro de téléphone secondaire |
| `bulk`     | boolean            | Indique si le colis fait partie d’un envoi groupé |
| `fragile`  | boolean            | Indique si le colis est fragile |
| `remarque` | string             | Commentaire ou note libre sur le colis |


**Body pour échange**

Dans le cas d’un échange, le champ service doit être défini à **"Échange"**
et il faut ajouter le code de l’ancien colis **(old_code)**.

Les autres champs obligatoires du colis (comme nom_complet, adresse, tel1, id_gov, etc.)
doivent également être présents dans la requête.

⚠️ Les champs **reason** et **apply_delivery** sont facultatifs dans le cas d’un échange.

```json
{
  "service": "Échange",
  "old_code": "CODE_ANCIEN_COLIS",
  "nom_complet": "Nom du destinataire",
  "adresse": "Adresse de livraison",
  "tel1": "12345678",
  "nombre_pieces": 1,
  "id_gov": 1
}
```

**Réponse succès :**
```json
{
  "status": "success",
  "message": "Colis créé avec succès.",
  "code": "010000010001",
  "etat_colis": "Créé"
}
```

### 4️⃣ Modification d’un colis

**🔒 PROTÉGÉ — Token requis**

**URL :** `PUT /api/colis/v1/{code}/modifier`

**Headers :**

```json
Authorization: Bearer {token}
Content-Type: application/json
```

**Body :**

```json
{
  "cod": 75.00,
  "adresse": "Nouvelle adresse",
  "tel1": "11111111",
  "remarque": "Nouvelle remarque"
}
```
**Réponse :**
```json
{
  "status": "success",
  "message": "Colis mis à jour avec succès."
}
```

### 5️⃣ Historique d’un colis
**🔒 PROTÉGÉ — Token requis**

**URL :** `GET /api/colis/v1/{code}/status`

**Headers :**

```json
Authorization: Bearer {token}
```
**Réponse :**

```json
{
  "status": "success",
  "colis_code": "010000010001",
  "etats": [
    {
      "etat": "Créé",
      "created_at": "2024-01-15 10:30:00"
    }
  ],
  "anomalies": [
    {
      "nom": "Adresse incorrecte",
      "commentaire": "Le destinataire a déménagé",
      "created_at": "2024-01-16 14:20:00"
    }
  ]
}
```

---

## 🗂️ 5.1 Liste des Anomalies possibles

| Nom de l’anomalie     | Code |
|------------------------|------|
| Téléphone fermé        | TF   |
| Pas de réponse         | PR   |
| Colis reporté          | RP   |
| Colis refusé           | ANN  |
| Adresse incorrecte     | AI   |
| Numéro incorrect       | NI   |
| Client non sérieux     | CLS  |


---

## 🚦 5.2 Liste des États possibles d’un colis

| # | Nom de l’état                | Description |
|---|------------------------------|--------------|
| 1 | Créé                         | Colis enregistré dans le système |
| 2 | En attente d’enlèvement      | En attente du livreur pour le pickup |
| 3 | Prêt à enlever               | Colis prêt à être récupéré |
| 4 | Enlevé                       | Colis pris par le livreur |
| 5 | Anomalie d’enlèvement        | Problème lors du pickup |
| 6 | Anomalie de retour           | Problème lors du retour |
| 7 | En stock                     | Colis dans l’agence |
| 8 | En cours de livraison        | En transit vers le destinataire |
| 9 | Livré                        | Colis remis au client |
| 10 | Livré Payé                  | Colis livré et payé |
| 11 | Planification retour        | En attente de retour planifié |
| 12 | Retourné                    | Colis retourné à l’expéditeur |
| 13 | En cours de transfert       | Colis en transit inter-agence |
| 14 | Anomalie de transfert       | Incident lors du transfert |
| 15 | À Vérifier                  | Statut en attente de validation |
| 16 | En attente d’échange        | Préparation d’un échange |
| 17 | Anomalie d’échange          | Problème lors d’un échange |
| 18 | Stock échange               | Colis d’échange en stock |
| 19 | Échange accepté             | Échange validé avec succès |



### 6️⃣ Exemples d’utilisation

**🔐 Obtenir un token**
```
curl -X POST https://youssel.online/api/login \
  -H "Content-Type: application/json" \
  -d '{"email": "test@example.com", "password": "password"}'
```
**📦 Créer un colis**
```
curl -X POST https://youssel.online/api/colis/v1/ajouter \
  -H "Authorization: Bearer VOTRE_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "nom_complet": "Mohamed Ali",
    "adresse": "12 Rue de la République",
    "tel1": "12345678",
    "nombre_pieces": 2,
    "service": "Livraison",
    "id_gov": 1
  }'

```
**🌍 Liste des gouvernorats (sans token)**
```
curl -X GET https://youssel.online/api/gouvernorat
```

### 7️⃣ Codes d’erreur

| Code | Description                |
|------|-----------------------------|
| 200  | Succès                     |
| 201  | Créé avec succès           |
| 401  | Token invalide ou manquant |
| 403  | Non autorisé               |
| 404  | Ressource non trouvée      |
| 422  | Erreur de validation       |
| 500  | Erreur serveur             |


### 8️⃣ Règles métier

* Seuls les colis **"Créé"** ou **"Prêt à enlever"** peuvent être modifiés.

* Les échanges nécessitent un ancien code valide.

* L’option Bulk nécessite des tarifs configurés.
