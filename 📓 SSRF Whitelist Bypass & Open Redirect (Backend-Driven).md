
## 🧠 Explication rapide

Une whitelist SSRF **n’est pas forcément vulnérable par intention**, mais souvent par **mauvaise validation**.

👉 Le problème central :

> le backend valide une **string (texte)**  
> alors que la requête part vers une **destination après parsing / normalisation**

➡️ Si ces deux visions divergent → **bypass possible**

---

# 🔍 Modèle mental SSRF (à garder)

Input utilisateur  
   ↓  
Filtre d’entrée (string)  
   ↓  
Parsing / canonicalisation  
   ↓  
Résolution DNS / IP  
   ↓  
Requête sortante réelle

👉 Une faille = **désalignement entre ces couches**

---

# 🔍 Observation

Cas étudié :

- whitelist sur host/domain
- validation naïve :
    - substring (`contains`)
    - prefix (`startsWith`)
- backend fait ensuite une requête HTTP

👉 Risque :

> la destination réelle ≠ celle validée

---

# hypothesisèses

### Hypothèse 1 — Match textuel naïf

Le filtre cherche seulement `"expected-host"` dans la string

➡️ possible :

- présent dans la string
- mais pas le vrai host

---

### Hypothèse 2 — Vérification de préfixe

https://expected-host...

➡️ mais parsing peut changer l’interprétation

---

### Hypothèse 3 — Décodage incohérent

- filtre → string brute
- backend → string décodée

➡️ divergence

---

### Hypothèse 4 — Pas de validation du host final

- pas de parsing strict
- pas de revalidation post-DNS

➡️ contrôle uniquement visuel

---

# 💣 Test discriminant (méthodologie propre)

## 🎯 Objectif

Identifier **à quelle couche ça casse**

---

## Test 1 — Match textuel

### Observation

- URL contenant expected-host
- mais structure ambiguë

### Signal

- accepté → requête tentée
- rejet → filtre plus strict

---

## Test 2 — Parsing réel

### Méthode

Changer la position de expected-host dans l’URL

### Observation

- comportements différents

### Interprétation

➡️ parsing incohérent

---

## Test 3 — Encoding

### Méthode

Comparer :

- brut
- encodé
- double encodé

### Observation

- acceptation différente

### Interprétation

➡️ mismatch validation / decoding

---

## Test 4 — Destination réelle

### Méthode

Utiliser destination contrôlée

### Observation

- DNS hit
- HTTP hit
- timing

### Interprétation

➡️ preuve SSRF réelle

---

# 🧪 Interprétation par technique

## 1️⃣ Userinfo (`@`)

expected-host@evil.com

- filtre → voit expected-host
- parsing → host = evil.com

🎯 Couche : parsing

---

## 2️⃣ Fragment (`#`)

evil.com#expected-host

- filtre → voit expected-host
- backend → ignore fragment

🎯 Couche : filtre d’entrée

---

## 3️⃣ Sous-domaine contrôlé

expected-host.attacker.com

- filtre → match substring
- réel → attacker.com

🎯 Couche : validation

---

## 4️⃣ Encoding

%2e → .

- filtre ≠ backend

🎯 Couche : canonicalisation

---

## 🧠 Insight clé

> Un bypass SSRF = rarement un payload unique  
> = souvent une **chaîne d’incohérences**

---

# ⚠️ Ce que ça prouve / ce que ça ne prouve pas

## ✔️ Prouve

- whitelist peut être contournable
- mismatch string vs parsing exploitable
- modèle multi-couches critique

---

## ❌ Ne prouve pas

- que la SSRF est exploitable
- que backend fait réellement une requête
- que toutes les variantes marchent

---

# ➜ Étape suivante

Construire une **matrice de test**

---

## Axe 1 — Type de validation

- substring
- prefix
- parsing réel
- revalidation

---

## Axe 2 — Signaux

- status code
- erreur
- timeout
- DNS / HTTP hit

---

## Axe 3 — Couche ciblée

- userinfo (@)
- fragment (#)
- subdomain
- encoding

---

---

# 🔥 PARTIE 2 — SSRF via Open Redirect

---

## 🧠 Explication rapide

👉 Tu ne bypass PAS la whitelist directement  
👉 Tu abuses d’un **service de confiance**

Whitelist OK → redirect → destination interdite

---

## 🔍 Observation

- backend valide domaine autorisé
- fait requête HTTP
- suit redirections (3xx)
- open redirect présent

---

## hypothesisèses

### H1 — Redirections suivies

✔️ nécessaire

---

### H2 — Validation pré-requête uniquement

✔️ pas de contrôle après redirect

---

### H3 — Redirect contrôlable

✔️ paramètre user-controlled

---

### H4 — Pas de filtrage final strict

✔️ ou partiel (ex: interne only)

---

# 💣 Test discriminant

---

## Test 1 — Redirection suivie ?

### Méthode

URL → redirect → serveur contrôlé

### Observation

- hit → suivi confirmé

---

## Test 2 — Validation initiale uniquement ?

### Méthode

direct → rejet  
via redirect → accepté

---

## Test 3 — Open redirect réelle ?

### Observation

302 Location: <controlled>

---

## Test 4 — Accès interne

### Méthode

redirect → IP interne

### Observation

- réponse
- timeout
- différence

---

# 🧪 Interprétation

input → whitelist OK  
       ↓  
trusted app (redirect)  
       ↓  
destination finale contrôlée  
       ↓  
SSRF réelle

---

# ⚠️ Ce que ça prouve / ce que ça ne prouve pas

## ✔️ Prouve

- redirection suivie
- contrôle final possible
- accès interne potentiel

---

## ❌ Ne prouve pas

- accès total réseau
- absence de firewall
- accès Internet

---

# ➜ Étape suivante (pro)

## 1. Cartographier chaîne

input → validation → redirect → requête finale

---

## 2. Tester limites

- IP interne / externe
- ports
- schémas
- profondeur redirect

---

## 3. Identifier cibles

- /admin
- API internes
- metadata cloud

---

## 4. Évaluer impact

- lecture
- action (POST)
- auth implicite

---

# 🧠 Mental model final

User input  
   ↓  
Whitelist OK  
   ↓  
Service autorisé (vulnérable)  
   ↓  
Redirect  
   ↓  
Destination interne  
   ↓  
SSRF

---

# 🔥 Insight bug bounty (très important)

> La whitelist ne protège que **la première requête**

👉 pas la chaîne complète

---

# 🧠 Résumé ultra clair

- Whitelist = fragile si basée sur string
- Parsing ≠ validation
- Redirect = pivot de confiance
- SSRF = contrôle de la destination finale




Parfait, on va ajouter une **couche fondamentale** : comprendre exactement **le pipeline SSRF côté backend**.  
C’est ça qui fait la différence entre “tester des payloads” et “comprendre pourquoi ça marche”.

---

# 🧠 Explication rapide

Une SSRF n’est **jamais directe**.

👉 Ton input traverse plusieurs étapes :

STRING → VALIDATION → PARSING → NORMALISATION → DNS → REQUÊTE

👉 Le bug apparaît quand :

> une étape “comprend” quelque chose de différent d’une autre

---

# 🔍 Observation

Dans ton lab, tu as vu :

- `/admin` ❌
- `http://192.168.0.12:8080/admin` ✅

👉 Donc :

- même input “logique”
- traitement différent selon **format + parsing**

---

# 🧩 Workflow SSRF complet (détaillé)

---

## 1️⃣ INPUT UTILISATEUR (STRING BRUTE)

### 🔍 Observation

stockApi=http://example.com

👉 Pour le backend, c’est juste :

"une suite de caractères"

---

### 🧠 Analogie

📦 Tu donnes une adresse écrite à la main :

> "12 rue machin Paris"

👉 À ce stade :

- personne ne sait si c’est valide
- c’est juste du texte

---

### ⚠️ Ce que ça implique

✔️ Le filtre travaille ici  
❌ Pas encore de logique réseau

---

---

## 2️⃣ FILTRE D’ENTRÉE (STRING MATCHING)

### 🔍 Observation

Code typique :

if "trusted.com" in url:  
    allow()

👉 Il regarde la **string brute**

---

### 🧠 Analogie

👮‍♂️ Agent de sécurité :

> “Je laisse passer si je vois écrit ‘trusted.com’”

👉 MAIS :

- il ne comprend pas l’adresse
- il regarde juste le texte

---

### 💣 Problème

http://trusted.com@evil.com

✔️ contient trusted.com  
❌ destination réelle = evil.com

---

### ⚠️ Ce que ça prouve / pas

✔️ validation faible possible  
❌ ne prouve pas encore SSRF

---

---

## 3️⃣ PARSING (INTERPRÉTATION STRUCTURELLE)

### 🔍 Observation

Le backend transforme :

"http://192.168.0.12:8080/admin"

➡️ en :

{  
  "scheme": "http",  
  "host": "192.168.0.12",  
  "port": 8080,  
  "path": "/admin"  
}

---

### 🧠 Analogie

📬 Facteur postal :

> Il ne lit pas la phrase  
> Il découpe :

- ville
- rue
- numéro

👉 Et livre selon ça

---

### 💣 Point critique

http://trusted.com@evil.com

Parsing :

- userinfo = trusted.com
- host = evil.com

👉 donc :  
➡️ requête vers evil.com

---

### ⚠️ Insight clé

> Le backend n’utilise JAMAIS la string brute  
> Il utilise la version parsée

---

---

## 4️⃣ CANONICALISATION / NORMALISATION

### 🔍 Observation

Avant la requête :

- décodage (`%2e`)
- simplification (`../`)
- nettoyage

---

### 🧠 Analogie

📦 Adresse mal écrite :

> "12 rue ../Victor Hugo"

👉 Le facteur corrige :

> "12 rue Victor Hugo"

---

### 💣 Problème

Filtre voit :

%2e%2e/admin

Backend utilise :

/admin

👉 divergence

---

### ⚠️ Ce que ça implique

✔️ double interprétation possible  
✔️ bypass encoding fréquent

---

---

## 5️⃣ RÉSOLUTION DNS

### 🔍 Observation

Le backend transforme :

example.com

➡️ en :

IP = 93.184.216.34

---

### 🧠 Analogie

📞 Tu cherches un nom dans un annuaire :

> “Donne-moi l’adresse de example.com”

---

### 💣 Point critique

Le filtre valide :

example.com

Mais DNS peut donner :

127.0.0.1

👉 DNS rebinding / manipulation possible

---

### ⚠️ Important

✔️ dernière étape avant réseau  
✔️ souvent non revalidée

---

---

## 6️⃣ REQUÊTE SORTANTE (RÉELLE)

### 🔍 Observation

Le serveur fait :

GET /admin HTTP/1.1  
Host: 192.168.0.12

---

### 🧠 Analogie

🚚 Livraison réelle :

> le colis part à l’adresse finale

---

### 💣 Point critique

C’est **la seule étape qui compte vraiment**

👉 Peu importe :

- ce que le filtre a vu
- ce que la string contenait

➡️ seule la destination finale compte

---

### ⚠️ Dans ton lab

✔️ 192.168.0.12 → accessible  
❌ google.com → erreur 500

👉 preuve :

- requête réellement envoyée
- mais filtrage réseau

---

---

# 🔥 Vue globale (ultra importante)

[1] STRING  
    ↓  
[2] FILTRE (texte)  
    ↓  
[3] PARSING (structure)  
    ↓  
[4] NORMALISATION  
    ↓  
[5] DNS  
    ↓  
[6] REQUÊTE RÉELLE

---

# 🧪 Interprétation

Une SSRF apparaît quand :

FILTRE ≠ PARSING ≠ DNS ≠ REQUÊTE

👉 désalignement

---

# ⚠️ Ce que ça prouve / ce que ça ne prouve pas

## ✔️ Ce que ça prouve

- SSRF = bug multi-couches
- parsing est central
- validation string seule = insuffisante

---

## ❌ Ce que ça ne prouve pas

- qu’un payload marche partout
- que toutes les étapes sont vulnérables
- que le backend ne revalide pas après

---

# ➜ Étape suivante (niveau expert)

Maintenant tu dois raisonner comme ça :

---

## 🧠 Question systématique

> “À quelle étape le backend prend la décision critique ?”

---

## 🎯 Objectif

Identifier :

|Étape|Question|
|---|---|
|Filtre|valide quoi ?|
|Parsing|interprète comment ?|
|Normalisation|modifie quoi ?|
|DNS|résout vers quoi ?|
|Requête|part où ?|

---

# 🧠 Résumé final

- STRING = ce que tu envoies
- FILTRE = ce que le dev croit voir
- PARSING = ce que le backend comprend
- DNS = où ça pointe réellement
- REQUÊTE = ce qui est exécuté

---

# 🔥 Insight bug bounty (très important)

> Tu n’exploites pas une URL  
> Tu exploites un **pipeline d’interprétation**