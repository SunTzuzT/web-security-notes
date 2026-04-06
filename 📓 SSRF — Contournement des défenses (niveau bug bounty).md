
## 🧠 Explication rapide

Une app vulnérable SSRF est souvent “protégée” par des filtres.

👉 Le problème :

> Le backend valide **la forme de l’URL (string)**  
> au lieu de valider **la destination réseau réelle**

💥 Résultat :

👉 Tu ne changes pas la cible  
👉 Tu changes **comment elle est écrite**

---

# 🔍 Modèle mental backend

User → envoie URL  
↓  
Backend :  
  - filtre (string)  
  - parse URL  
  - decode / canonicalise  
  - résout DNS  
  - fait la requête  
↓  
Service cible

---

## 💥 Point clé

> La vulnérabilité = **décalage entre**

- ce que le filtre voit
- ce que la requête appelle réellement

---

# 🧩 Défenses SSRF courantes

## 1. Blacklist (fragile)

### 🔍 Observation

Filtre bloque :

- `127.0.0.1`
- `localhost`
- `/admin`

### 🧠 Hypothèse

- comparaison textuelle
- pas de canonicalisation

### 💣 Impact

👉 contournable facilement

---

## 2. Allowlist (plus dangereux)

### 🔍 Observation

Autorise :

- `example.com`

### 🧠 Hypothèse

- validation partielle
- parsing ambigu

### 💣 Impact

👉 bypass via parsing / DNS / redirection

---

## 3. Filtre de schéma

http / https only

👉 insuffisant

---

## 4. Blocage IP privées

127.0.0.1  
10.0.0.0/8

👉 souvent mal implémenté

---

# 💣 Techniques de bypass (logique attaquant)

---

## 1. Obfuscation IP

### 🔍 Observation

`127.0.0.1` bloqué

### 🧠 Hypothèse

filtre basé sur string

### 💣 Tests

2130706433  
127.1  
0x7f000001  
017700000001

### 🧪 Interprétation

backend voit ≠  
réseau résout = localhost

---

## 2. Encoding / double encoding

### 🔍 Observation

`/admin` bloqué

### 🧠 Hypothèses

- decode avant filtre
- double interprétation

### 💣 Tests

/%61dmin      → test  
/%2561dmin    → test discriminant

### 🧪 Interprétation

Si :

%61 → bloqué  
%2561 → passe

👉 pipeline :

decode 1 → filtre → decode 2

---

## 3. Parsing URL (incohérences)

### 🔍 Observation

le backend interprète mal l’URL

### 💣 Tests

localhost@target  
admin@target

### 🧪 Interprétation

- filtre voit `localhost`
- requête part ailleurs

---

## 4. Redirection contrôlée

### 🔍 Observation

validation initiale seulement

### 💣 Setup

attacker.com → 302 → 127.0.0.1

### 🧪 Interprétation

backend ne revalide pas

---

## 5. DNS bypass

### 🔍 Observation

validation sur hostname

### 💣 Test

domaine contrôlé → IP interne

### 🧪 Interprétation

résolution finale ≠ validation initiale

---

## 6. Metadata cloud

### 🔍 Observation

localhost filtré, metadata oubliée

### 💣 Impact

- credentials AWS/GCP
- pivot infra

---

# 🧪 Méthodologie attaquant (critique)

---

## 🎯 Question centrale

> Le backend valide :

- la string ? ❌
- ou la destination réelle ? ✔️

---

## 🔁 Workflow propre

### 1. Observation

- blocked ?
- timeout ?
- 500 ?
- 302 ?

---

### 2. Hypothèses

- filtre string ?
- decode avant filtre ?
- DNS check ?
- redirect follow ?

---

### 3. Tests discriminants

|Test|Objectif|
|---|---|
|IP alternative|bypass host|
|encoding|bypass path|
|double encoding|test decode|
|redirect|test revalidation|
|DNS|test résolution|

---

### 4. Interprétation

|Réponse|Lecture|
|---|---|
|blocked|filtre|
|timeout|réseau|
|500|appel backend réel|
|200|accès réussi|

---

# 🧠 Lab PortSwigger — Lecture propre

---

## 🔍 Observation

127.0.0.1 → bloqué  
127.1 → passe  
127.1/admin → bloqué  
127.1/%2561dmin → passe

---

## hypothesesèses

- filtre string host
- filtre string path
- decode avant filtre
- double decode après

---

## 💣 Test discriminant

/%61dmin → bloqué  
/%2561dmin → passe

---

## 🧪 Interprétation

decode 1 → filtre → decode 2

👉 bypass canonicalisation

---

## ⚠️ Ce que ça prouve

- filtre non robuste
- absence de normalisation
- incohérence parsing / exécution

---

## ➜ Étape suivante

/admin/delete?username=carlos

- gérer :

302 → suivre redirection

---

# ⚠️ Points critiques bug bounty

---

## 1. “Blocked” ≠ fin

👉 = preuve que :

- fonctionnalité SSRF existe
- validation est attaquable

---

## 2. Toujours raisonner en couches

|Couche|Question|
|---|---|
|filtre|que bloque-t-il ?|
|parsing|comment il lit l’URL ?|
|decode|combien de fois ?|
|DNS|où ça pointe vraiment ?|
|HTTP client|que fait-il vraiment ?|

---

## 3. Les bugs viennent des incohérences

👉 entre :

- filtre custom
- parser URL
- DNS resolver
- client HTTP

---

## 4. Objectif réel

Pas juste “réponse différente”

Mais :

- accès localhost
- service interne
- admin panel
- metadata cloud
- action backend

---

## 5. Impact réel

- fuite secrets
- credentials cloud
- pivot interne
- exécution actions backend
- escalation

---

# 🎯 Résumé mental final

Défense faible :  
"je vérifie l’URL telle qu’écrite"  
  
Défense correcte :  
"je normalise, je résous, je valide la destination réelle"

---

## 🔥 Phrase clé (à retenir)

> SSRF = exploiter un décalage entre  
> **validation de l’URL** et **destination réelle**