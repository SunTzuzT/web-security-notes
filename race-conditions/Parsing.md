
# 🔥 1. Modèle mental global

HTTP request  
→ parsing (type réel backend)  
→ état (session / DB / cache)  
→ check  
→ write  
→ usage final (email / action)

---

## 🔑 Insight clé

> Le bug = incohérence entre **ce qui est écrit, ce qui est lu, et quand**

---

# 🧠 2. Race Condition (fondation)

## 📖 Définition

Deux requêtes manipulent le **même état** au même moment

Req A → read état X  
Req B → read état X  
Req A → write  
Req B → write

➡️ incohérence

---

## 🎯 Conditions nécessaires

- état partagé (DB / session / cache)
- non atomicité
- exécution concurrente

---

# 🧠 3. TOCTOU

CHECK → (window) → USE

➡️ état peut changer entre les deux

---

## 🔑 Insight

> si check ≠ moment d’utilisation → exploitable

---

# 🧠 4. Single Endpoint Race (IMPORTANT)

## 📖 Définition

➡️ même endpoint  
➡️ même session  
➡️ mêmes variables backend  
➡️ mais requêtes concurrentes

---

## ⚙️ Pattern backend

session['user'] = X  
session['token'] = Y  
send email (async)

---

## 💣 Fail

Req A → user = attacker  
Req B → user = victim

A génère token  
B écrase user  
email utilise état B

➡️ token attacker appliqué à victim

---

## 🧠 Analogie

Tableau blanc :

- A écrit
- B écrase
- quelqu’un lit après

---

# 🧪 LAB — Change Email (Single Endpoint)

---

## 🔍 Observation

- endpoint unique `/change-email`
- état unique `pending_email`
- overwrite à chaque requête
- email envoyé async

---

## 💣 Test discriminant

Même session :

Req A → email attacker  
Req B → email carlos  
(parallèle)

---

## 🧪 Interprétation

Si :

- mail reçu chez toi
- contenu = carlos

➡️ désynchronisation confirmée

---

## 🔑 Insight

- source = DB/session mutable
- pas d’atomicité
- async = surface critique

---

# 🧠 5. Timestamp Token (crypto faible)

---

## 🔍 Observation

- tokens différents
- longueur fixe
- dépendance temporelle

---

## ⚙️ Backend réel

token = hash(timestamp)

---

## 💣 Exploit

Req A → wiener  
Req B → carlos  
(parallèle)

➡️ même timestamp  
➡️ même token

---

## 🧪 Interprétation

- token non lié au user
- collision possible

---

## 🎯 Résultat

/confirm?user=carlos&token=TON_TOKEN

➡️ account takeover

---

# 🧠 6. Session Lock (PHP)

---

## 🔍 Observation

- 1 session = 1 requête
- traitement séquentiel

---

## 🧪 Détection

Req A → 200ms  
Req B → 400ms

➡️ lock

---

## 💣 Bypass

- nouvelle session
- supprimer cookie
- GET pour régénérer session + CSRF

---

## 🔑 Insight

> un race qui “ne marche pas” = souvent un lock

---

# 🧠 7. Partial Object Construction

---

## ⚙️ Backend

1. create user  
2. set api_key

---

## ⚠️ Fenêtre critique

user existe  
api_key = NULL

---

## 💣 Exploit

api_key[]=

➡️ devient `[]`

---

## 🧪 Interprétation

[] == NULL → TRUE (selon logique faible)

➡️ bypass

---

## 🧠 Analogie

Compte créé sans mot de passe

---

# 🧠 8. Parsing (fondation exploitation)

---

## 📖 Définition

HTTP → transformé en type backend

---

## 🔍 Exemples

param=abc        → "abc"  
param[]=abc      → ["abc"]  
param[]=         → []  
param[key]       → {key: nil}

---

## 💣 Exploit

Backend attend :

string

Tu envoies :

array / object / empty

➡️ mismatch → validation cassée

---

## 🔑 Insight

> tu attaques la **compréhension backend**, pas la valeur

---

# 🧠 9. Modes d’envoi (CRITIQUE)

---

## 🔹 Single connection

- requêtes très proches
- peu de jitter
- timing précis

👉 utile : fenêtres très courtes

---

## 🔹 Separate connections

- isolation
- utile pour analyser comportement

---

## 🔹 Parallèle

- exécution simultanée backend

👉 indispensable pour race

---

## 🔑 Différence clé

|Mode|Objectif|
|---|---|
|Single|précision timing|
|Separate|observation|
|Parallèle|chevauchement réel|

---

# 🧠 10. Grille mentale universelle (CORE)

---

## 1. Qu’attend le backend ?

(type, format)

---

## 2. Quelle hypothèse ?

(ex: “toujours string”)

---

## 3. Source de vérité ?

(DB / session / cache)

---

## 4. Quand check vs write ?

(TOCTOU)

---

## 5. Puis-je changer la représentation ?

(array / null / empty)

---

# 🧠 11. Automatisme lecture requête

---

Rôle métier  
→ champs sensibles  
→ décision backend  
→ hypothèse implicite  
→ test minimal

---

# 🧠 12. Quand tester parsing ?

---

## ✅ Oui si :

- auth / token / api_key
- ID / userId
- flags (isAdmin)
- paramètres optionnels
- logique métier

---

## ❌ Non si :

- tracking
- read-only simple
- aucun impact backend

---

# 🧠 13. Signaux faibles (ULTRA IMPORTANT)

- résultats non déterministes
- tokens invalides aléatoires
- incohérences email / action
- comportement différent même input

➡️ souvent race / parsing

---

# 🧠 14. Défense (lecture inversée)

---

Dev veut :

- atomicité
- 1 source de vérité
- pas d’état intermédiaire

---

Toi tu cherches :

- état partiel
- multi source
- non atomique
- async

---

# 🎯 TL;DR GLOBAL

- parsing → change interprétation
- race → change timing
- TOCTOU → change ordre
- partial → exploite état incomplet

➡️ exploit = incohérence backend

---

# 🧠 Phrase finale (niveau élite)

> “Je ne cherche pas un payload.  
> Je cherche où le backend peut se tromper.”
