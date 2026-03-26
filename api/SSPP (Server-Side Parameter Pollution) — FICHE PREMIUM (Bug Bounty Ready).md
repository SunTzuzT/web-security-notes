
# 🎯 🧩 Définition (version terrain)

> **SSPP = tu contrôles comment le backend construit UNE AUTRE requête backend.**

👉 Tu ne modifies pas juste un paramètre.  
👉 Tu modifies **la communication interne entre services**.

---

# 🧠 💡 ANALOGIE MAÎTRE (à retenir absolument)

### 🏨 Le réceptionniste naïf

- Toi → client
- API publique → réceptionniste
- API interne → service VIP inaccessible

---

👉 Tu dis :

> “Je veux Peter”

---

👉 Le réceptionniste envoie :

```
chercher(name=peter, field=email)
```

---

👉 Mais tu dis :

```
peter + "field=reset_token"
```

---

💥 Résultat :

```
chercher(name=peter, field=reset_token)
```

---

👉 Tu viens de manipuler une **requête interne invisible**.

---

# 🏗️ 🧠 Architecture réelle (vision pro)

```
[Client]
   ↓
[Frontend JS]
   ↓
[API publique]
   ↓  (⚠️ zone vulnérable SSPP)
[API interne]
   ↓
[BDD / services]
```

---

👉 SSPP se situe ici :

> 🔥 **entre API publique et API interne**

---

# ⚔️ 🧠 LES 3 TYPES DE SSPP

---

# 🟦 1. SSPP — Query String

---

## 📍 Où ?

Dans l’URL :

```
?param=value&param2=value2
```

---

## 🔑 Rappel

|élément|rôle|
|---|---|
|`&`|ajoute un paramètre|
|`#`|coupe la requête|

---

## 🧪 Exemple réel

```
GET /userSearch?name=peter
```

↓

```
GET /users/search?name=peter&field=email
```

---

## 💥 Injection

```
name=peter%26field=reset_token%23
```

↓

```
name=peter&field=reset_token#
```

↓

```
/users/search?name=peter&field=reset_token
```

---

## 🎯 Impact

- fuite de données
- token reset
- bypass logique

---

## 🧠 ANALOGIE

👉 Tu ajoutes une option dans une commande interne.

---

## ⚠️ PIÈGE

👉 croire que `#` agit côté client uniquement  
👉 ici il casse la requête backend

---

---

# 🟥 2. SSPP — Path (REST)

---

## 📍 Où ?

Dans le chemin :

```
/api/users/123
```

---

## 🧪 Exemple

```
GET /edit_profile.php?name=peter
```

↓

```
GET /api/private/users/peter
```

---

## 💥 Injection

```
name=peter%2f..%2fadmin
```

↓

```
/api/private/users/peter/../admin
```

↓

```
/api/private/users/admin
```

---

## 🎯 Impact

- accès autre utilisateur
- BOLA
- bypass contrôle accès

---

## 🧠 ANALOGIE

👉 Tu changes de dossier dans un système de fichiers

```
peter → ../admin
```

---

## ⚠️ PIÈGE

👉 penser que c’est juste du path traversal  
👉 ici c’est **injection dans un path backend**

---

---

# 🟩 3. SSPP — JSON / Structured Data

---

# 📍 Où ?

Dans le **BODY** d’une requête HTTP :

```
Content-Type: application/json
```

---

## 🧠 C’EST QUOI JSON ?

👉 Format structuré clé/valeur :

```
{"name":"peter"}
```

---

👉 Utilisé dans :

- API REST
- mobile apps
- GraphQL
- microservices

---

## 🧪 Exemple

```
POST /myaccount
{"name":"peter"}
```

↓

```
PATCH /users/7312/update
{"name":"peter"}
```

---

## 💥 Injection

```
{"name":"peter\",\"role\":\"admin"}
```

↓

```
{"name":"peter","role":"admin"}
```

---

## 🎯 Impact

- privilege escalation
- admin access
- business logic abuse

---

## 🧠 ANALOGIE

👉 Tu écris dans un formulaire :

```
Nom = peter", role="admin
```

---

👉 Le système comprend :

> “ajoute un champ role=admin”

---

## ⚠️ PIÈGE

👉 croire que c’est du Mass Assignment  
👉 ici tu **casses la structure JSON backend**

---

---

# 🔗 🧠 CONNEXION AVEC LES AUTRES FAILLES

---

## 🔐 AUTHENTICATION

👉 SSPP → récupérer reset_token  
👉 ⇒ account takeover

---

## 🔓 ACCESS CONTROL (BOLA)

👉 SSPP → changer ID / user  
👉 ⇒ accès autre compte

---

## 💰 BUSINESS LOGIC

👉 SSPP → injecter :

```
"price":1
"coupon":"VALID"
```

👉 ⇒ fraude

---

## 📦 MASS ASSIGNMENT

|   |   |
|---|---|
|SSPP|Mass Assignment|
|injection backend|champs acceptés|
|modifie requête interne|modifie input direct|

---

## 💉 SQL INJECTION (analogie)

👉 même logique :

|   |   |
|---|---|
|SQLi|SSPP|
|casse SQL|casse HTTP/JSON|
|DB|API interne|

---

---

# 🌐 🔍 JAVASCRIPT = ORACLE 🔥

---

## 📍 Où le trouver ?

- Burp → HTTP history → `/static/js/...`
- DevTools → Network / Sources

---

## 🔍 Quoi chercher ?

```
token
reset
fetch
endpoint
```

---

## 💥 Exemple clé

```
window.location.href = `/forgot-password?reset_token=${resetToken}`;
```

---

## 🧠 Traduction

👉 endpoint = `/forgot-password`  
👉 param = `reset_token`

---

👉 C’est comme une **doc API cachée**

---

---

# 🧠 🧭 MÉTHODOLOGIE BUG BOUNTY

---

## 🔥 Étape 1 — détecter

- erreurs backend
- comportements étranges
- transformations input

---

## 🔥 Étape 2 — injecter

|   |   |
|---|---|
|Type|Payload|
|Query|`%26`, `%23`|
|Path|`../`|
|JSON|`","admin":true`|

---

## 🔥 Étape 3 — observer

- erreurs
- réponses
- structure

---

## 🔥 Étape 4 — comprendre

👉 “comment le backend reconstruit la requête ?”

---

## 🔥 Étape 5 — exploiter

- param caché
- valeur sensible
- pivot logique

---

---

# 🚨 🧠 PIÈGES CLASSIQUES

---

❌ tester au hasard sans lire les erreurs  
❌ ignorer le JS  
❌ ne pas comprendre le format (query vs JSON)  
❌ penser “c’est juste du input”

---

👉 SSPP ≠ input  
👉 SSPP = **construction backend**

---

---

# 🧠 🧩 SIGNES D’UNE SSPP

- erreurs parlant de paramètres inconnus
- comportement différent avec `%26` ou `%23`
- JSON modifié
- réponses inattendues
- indices dans JS

---

---

# 🧠 🏁 RÉSUMÉ FINAL

---

👉 SSPP = contrôler la requête backend

---

👉 3 variantes :

- Query → `&`, `#`
- Path → `../`
- JSON → casser structure

---

👉 outils :

- Burp (manuel)
- JS (indice)
- Intruder (énumération)

---

👉 impact :

- auth bypass
- BOLA
- business logic
- data leak

---

# 💬 🧠 PHRASE À RETENIR

> 💥 **SSPP = je manipule la façon dont le serveur parle à lui-même.**

---

# 🚀 NIVEAU ATTEINT

👉 Si tu maîtrises cette fiche :

> 🔥 tu es capable de trouver des vraies failles API en bug bounty

---
