
## 📌 Définition

> **Server-Side Parameter Pollution (SSPP)** = vulnérabilité où une application backend réutilise des **entrées utilisateur** pour construire une requête vers une **API interne**, sans validation stricte.

💥 Résultat :

- injection de paramètres
- modification de logique backend
- accès à des données internes

---

## 🧩 Vision globale (architecture)

```
[Utilisateur]
     ↓
[API publique / Backend]
     ↓ (reconstruction requête)
[API interne]
     ↓
[Base de données]
```

👉 Le bug est ici :

```
internal_request = base_query + user_input
```

---

## 🧠 Analogie clé (pivot)

👉 C’est un **pivot applicatif**

- Tu ne peux PAS accéder à l’API interne directement
- Tu passes par l’API publique
- Tu influences ce qu’elle envoie

💥 Comme en réseau :

```
Internet → serveur exposé → réseau interne
```

---

## ⚔️ Différence avec Mass Assignment

||Mass Assignment|SSPP|
|---|---|---|
|Cible|données envoyées|requête interne|
|Action|ajouter champs|modifier requête backend|
|Type|direct|indirect|

---

## 🔐 URL Encoding (CRUCIAL)

👉 Certains caractères sont interdits en brut → encodés

|   |   |   |
|---|---|---|
|Caractère|Encodé|Rôle|
|`#`|`%23`|coupe la requête|
|`&`|`%26`|ajoute paramètre|
|`=`|`%3D`|assigne valeur|

---

### ⚙️ Cycle réel

```
1. Tu envoies : peter%26email=foo
2. Serveur decode → peter&email=foo
3. Backend reconstruit la requête
```

💥 L’injection se fait **après décodage**

---

## 🧪 Techniques d’exploitation

---

### 1️⃣ Troncature (`#`)

```
name=peter%23foo
```

↓

```
name=peter#foo&publicProfile=true
```

👉 `#` ignore le reste

💥 Permet de supprimer :

- filtres
- flags internes (`publicProfile=true`)

---

### 2️⃣ Injection paramètre (`&`)

```
name=peter%26foo=bar
```

↓

```
name=peter&foo=bar&publicProfile=true
```

👉 Ajout d’un paramètre

---

### 3️⃣ Injection paramètre valide

```
name=peter%26email=test@test.com
```

👉 Exploite paramètres backend réels

💥 Objectif :

- filtrer autrement
- accéder à data

---

### 4️⃣ Remplacement paramètre

```
name=peter%26name=admin
```

↓

```
name=peter&name=admin
```

---

👉 Dépend du backend :

|   |   |
|---|---|
|Tech|Comportement|
|PHP|dernier param|
|Node.js|premier|
|ASP.NET|combine|

💥 Peut mener à :

- account takeover
- bypass auth

---

## 🔍 Méthodologie terrain (Burp mindset)

---

### Step 1 — Identifier entrée utilisateur

- query params
- JSON body
- headers
- path

---

### Step 2 — Tester injection

```
%23 → troncature
%26 → ajout paramètre
duplicate param → override
```

---

### Step 3 — Observer

- changement réponse
- données en plus
- erreurs
- comportement incohérent

---

### Step 4 — Énumérer paramètres

Tester :

```
admin=true
role=admin
email=
userId=
debug=true
internal=true
```

---

## 🚨 Signes de vulnérabilité

- API agit comme proxy
- paramètres inconnus acceptés
- réponse influencée par paramètres ajoutés
- incohérences backend

---

## ⚠️ Pièges importants

❌ oublier l’encodage (`%23`, `%26`)  
❌ confondre avec SQL injection  
❌ tester sans observer réponse  
❌ ne pas comprendre le flow backend

---

## 🧠 Analogies pédagogiques

---

### 🧾 SQL Injection

👉 tu modifies une requête SQL

---

### 🧵 SSPP

👉 tu modifies une requête HTTP interne

---

### 🚗 Uber

- toi = client
- API publique = chauffeur Uber
- API interne = système interne Uber

💥 Tu modifies les instructions envoyées au système

---

## 🎯 Résumé ultra clair

👉 Tu ne hacks pas l’API directement  
👉 Tu hacks **la manière dont elle parle à une autre API**

---

## 💥 Impact réel

- accès données privées
- bypass auth
- modification logique métier
- accès API interne

---

## 🚀 Niveau bug bounty

💰 Très intéressant si combiné avec :

- IDOR
- API testing
- business logic

---

## 🧠 Mentalité à adopter

> “Est-ce que cette requête est reconstruite côté serveur ?”

---

👉 Si oui → surface d’attaque

---

## 📌 À retenir

- injection après décodage
- contrôle indirect du backend
- pivot applicatif
- logique > technique brute

---
