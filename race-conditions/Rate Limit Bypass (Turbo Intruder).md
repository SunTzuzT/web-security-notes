
## 🎯 Concept clé

> Une race condition exploitable existe quand :

CHECK (autorisation / limite)  
→ THEN  
WRITE (mise à jour état)

👉 Si non atomique → plusieurs requêtes passent avant le verrou

---

## 🔍 Observation

- Login protégé par **rate limit**
- Limite appliquée **par username (carlos)**
- Backend fait :

check attempts < limit  
→ process login  
→ increment attempts

👉 fenêtre entre check et write exploitable

---

## ⚙️ Mécanisme réel (TOCTOU)

t0 → 10 requêtes arrivent  
t1 → check attempts = 0 → OK pour toutes  
t2 → password check  
t3 → increment attempts → trop tard

👉 plusieurs tentatives passent avant blocage

---

## 💣 Exploit technique

### Conditions obligatoires

- HTTP/2 ✅
- 1 seule connexion (`concurrentConnections=1`)
- multiplexage → synchronisation parfaite

---

## 🚀 Turbo Intruder – Single Packet

engine = RequestEngine(  
    endpoint=target.endpoint,  
    concurrentConnections=1,  
    engine=Engine.BURP2  
)  
  
for password in passwords:  
    engine.queue(target.req, password, gate='1')  
  
engine.openGate('1')

---

## 🧪 Lecture des réponses

|Signal|Signification|
|---|---|
|200 (normal)|password incorrect|
|4185 / 4186|rate limit|
|302 🔥|SUCCESS|

👉 **Seul signal fiable = 302**

---

## ⚠️ Pièges critiques

### 1. Mauvaise requête

❌ `GET /login`  
✅ `POST /login`

---

### 2. Mauvais timing

❌ lancer après essais  
✅ lancer DIRECT après reset

---

### 3. Mauvais endpoint state

- compteur déjà élevé → race impossible
- backend bloque AVANT check

---

### 4. Faux signal UI

UI → “too many attempts” ❌  
Turbo Intruder → vérité backend ✅

---

## ⚡ Fenêtre de race

👉 limitée (~3–5 requêtes utiles)

- - de requêtes ≠ + d’efficacité
- bruit ↑
- blocage ↑

---

## 🧠 Lecture backend avancée

Le système fait probablement :

if attempts > limit:  
    block

👉 mais **après incrément**

Donc :

- bypass possible pendant execution
- mais état global pollué ensuite

---

## 🔄 Workflow optimal (lab)

RESET LAB  
→ aucune requête  
→ Turbo Intruder DIRECT  
→ 1 seul burst (wordlist complète)  
→ chercher 302  
→ login  
→ /admin  
→ delete carlos

---

## 🔐 CSRF comportement

🔍 Observation

- refresh → token invalide
- back → fonctionne

👉 CSRF lié à la session/page

🎯 règle :

- utiliser page fraîche
- ou revenir en arrière

---

## 🧩 Placeholders

|Outil|Placeholder|
|---|---|
|Intruder|`§pass§`|
|Turbo Intruder|`%s`|

👉 `%s` injecte automatiquement les payloads

---

## ⚡ Insight élite

> Ce n’est PAS une brute force classique

C’est :

timing exploitation > volume

---

## 🧠 Pattern bug bounty réel

Cherche :

- OTP limité
- coupon unique
- crédit consommable
- bonus 1 fois
- stock limité
- actions idempotentes

👉 toujours poser :

> “le check et le write sont-ils séparés ?”

---

## 🧨 Résumé brutal

- race = fenêtre entre check et write
- Turbo Intruder = outil de timing
- HTTP/2 = clé
- 302 = succès
- reset = nécessaire
- UI ≠ vérité
- timing > brute force






# 🔍 1. Observation (phase reconnaissance)

### Test simple

- 3 mauvais passwords → OK
- 4e → **account locked**

👉 protection active

---

### Test avec autre user

- réponse = `Invalid username or password`

👉 déduction :

rate limit basé sur USERNAME (pas session)

---

# 🧠 1.5 ⚠️ Pourquoi vérifier que le compteur est côté serveur

## 🔍 Observation

On cherche à savoir **où est stocké le compteur** :

- côté client ? (cookie / JS)
- côté serveur ? (DB / cache / mémoire)

---

## 💣 Test discriminant (minimal)

👉 Change :

- navigateur / session / cookie

ET reteste le login

---

## 🧪 Interprétation

### Cas 1 — compteur côté client ❌

- reset cookie → reset compteur
- bypass trivial

👉 PAS intéressant en bug bounty (low impact)

---

### Cas 2 — compteur côté serveur ✅

- reset navigateur → toujours bloqué
- limite persiste

👉 protection réelle

---

## 🎯 Pourquoi c’est CRITIQUE pour la race

Race condition = compétition entre requêtes côté serveur

👉 Donc :

- si le compteur est côté client → PAS de race exploitable
- si le compteur est côté serveur → possibilité de :

plusieurs requêtes lisent le même état AVANT mise à jour

---

## 🧠 Lecture backend

Cas vulnérable :

READ attempts = 0  
READ attempts = 0  
READ attempts = 0  
→ toutes passent  
  
WRITE attempts = 1  
WRITE attempts = 2  
WRITE attempts = 3

👉 trop tard

---

## ⚠️ Insight clé

Une race condition n’existe que sur un état partagé serveur

👉 pas sur un état local client

---

## 🔥 Conclusion

👉 Vérifier que le compteur est côté serveur permet de confirmer :

- qu’il y a un **état global partagé**
- donc une **fenêtre de concurrence exploitable**

---

# 🧠 Déduction backend

compteur stocké côté serveur  
(par user)

---

## ⚠️ Hypothèse critique

CHECK (attempts < limit)  
→ THEN  
WRITE (increment attempts)

👉 possible fenêtre entre les deux

---

# 🧪 2. Évaluation du comportement (preuve)

## Étape A — Séquentiel

- Repeater
- envoi un par un

👉 blocage rapide → comportement normal

---

## Étape B — Parallèle

- envoi simultané

👉 résultat :

plus de requêtes passent que la limite

---

## 🧠 Interprétation

plusieurs requêtes passent AVANT mise à jour du compteur

👉 race confirmée

---

# ⚙️ 3. Exploit avec Turbo Intruder

---

## 🔑 Conditions

- HTTP/2
- BURP2 engine
- concurrentConnections = 1
- gate

---

## 🧩 Requête

POST /login  
username=carlos&password=%s

---

## 🚀 Script officiel

def queueRequests(target, wordlists):  
  
    engine = RequestEngine(  
        endpoint=target.endpoint,  
        concurrentConnections=1,  
        engine=Engine.BURP2  
    )  
      
    passwords = wordlists.clipboard  
      
    for password in passwords:  
        engine.queue(target.req, password, gate='1')  
      
    engine.openGate('1')  
  
  
def handleResponse(req, interesting):  
    table.add(req)

---

# 🧪 4. Analyse

|Code|Signification|
|---|---|
|200|mauvais password|
|4186|rate limit|
|302 🔥|succès|

---

# ⚡ 5. Mécanisme réel

t0 → requêtes arrivent  
t1 → check = OK  
t2 → traitement login  
t3 → write (trop tard)

---

# 🔄 6. Workflow optimal

RESET  
→ aucune requête  
→ Turbo Intruder DIRECT  
→ 1 burst  
→ chercher 302  
→ login  
→ /admin  
→ delete

---

# 🔥 Résumé brutal

- vérifier stockage serveur = étape clé
- race = état partagé + non atomique
- bypass = timing
- brute force = conséquence

---

## 🧠 Mental model final

Si une limite dépend d’un état serveur partagé  
→ vérifier si ce state est atomique  
→ sinon → race exploitable
