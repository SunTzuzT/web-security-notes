# 🔹 1. C’est quoi une API ?

## 📖 Définition

> 💥 Une API = une interface qui permet de parler directement au backend

---

## 🧠 Analogie

👉 Site = restaurant  
👉 API = cuisine

- Front = menu
- API = commande envoyée en cuisine
- Backend = cuisinier

💥 En bug bounty :

> tu vas directement parler au cuisinier

---

## 🔁 Flow réel

```
Navigateur → HTTP → API → backend → base de données → réponse
```

---

# 🔹 2. Structure d’une requête HTTP

## Exemple

```
POST /api/account/update HTTP/1.1
Host: app.com
Authorization: Bearer TOKEN
Content-Type: application/json

{
  "email": "test@test.com"
}
```

---

## 🧩 Les parties

|Partie|Rôle|
|---|---|
|méthode|action|
|endpoint|cible|
|headers|contexte|
|body|données|

---

# 🔹 3. 🔥 C’est quoi le CONTEXTE ?

## 📖 Définition simple

> 💥 Le contexte = les informations autour de ta requête qui influencent la décision du backend

---

## 🧠 Analogie

👉 Tu arrives dans un restaurant :

- tu dis “je veux une pizza”
- MAIS :
    - tu es VIP ?
    - tu es staff ?
    - tu es client normal ?

👉 la réponse change

💥 ça = le contexte

---

## Exemple concret

```
GET /api/account HTTP/1.1
Authorization: Bearer USER_TOKEN
```

👉 réponse :

```
{"role":"user"}
```

---

```
GET /api/account HTTP/1.1
Authorization: Bearer ADMIN_TOKEN
```

👉 réponse :

```
{"role":"admin"}
```

💥 même requête  
💥 contexte différent → résultat différent

---

## ⚠️ Le contexte inclut :

- Authorization (token)
- cookies
- headers (`X-User-Id`)
- IP (parfois)
- rôle
- session

---

# 🔹 4. Observable vs Contrôlable vs Injectable

## 👀 Observable

👉 ce que tu vois

```
{
  "userId": 1,
  "role": "user",
  "balance": 0
}
```

---

## 🎮 Contrôlable

👉 ce que tu peux modifier

```
POST /api/update
Authorization: Bearer TOKEN
Content-Type: application/json

{"email":"a@test.com"}
```

---

## 💉 Injectable

👉 ce qui peut influencer le backend

```
{"email":"test@test.com"}
```

---

## ⚠️ ERREUR

❌ voir ≠ modifier  
✔️ seul ce que tu envoies = testable

---

# 🔹 5. Parsing (CRUCIAL)

## 📖

> 💥 Parsing = comment le backend comprend tes données

---

## Exemple JSON

```
Content-Type: application/json

{"id":10}
```

---

## Exemple form

```
Content-Type: application/x-www-form-urlencoded

id=10
```

---

👉 même donnée  
👉 parsing différent

---

## 💥 Danger

```
Content-Type: application/json
{"role":"admin"}
```

👉 bloqué

---

```
Content-Type: application/x-www-form-urlencoded
role=admin
```

👉 accepté → faille

---

# 🔹 6. Méthodes HTTP

## Exemple

```
GET /api/tasks
```

```
POST /api/tasks
```

```
DELETE /api/tasks/1
```

---

## 💥 Test

```
OPTIONS /api/tasks
```

👉 réponse :

```
Allow: GET, POST, DELETE
```

---

# 🔹 7. Content-Type

## Exemple

### JSON

```
Content-Type: application/json

{"id":1}
```

---

### FORM

```
Content-Type: application/x-www-form-urlencoded

id=1
```

---

### XML

```
Content-Type: application/xml

<user><id>1</id></user>
```

---

## 💥 À retenir

> 💥 format différent = logique différente

---

# 🔹 8. Headers (souvent oubliés)

## Exemple vulnérable

```
GET /api/account HTTP/1.1
Authorization: Bearer USER_TOKEN
X-User-Id: 999
```

👉 réponse :

```
{"userId":999,"role":"admin"}
```

💥 IDOR via header

---

## Headers à tester

```
X-User-Id
X-Role
X-Admin
X-Forwarded-For
```

---

# 🔹 9. Workflow terrain (ULTRA IMPORTANT)

## 🔹 Étape 1 — comprendre

```
POST /api/account/update
```

👉 action = update

---

## 🔹 Étape 2 — identifier

- ID ?
- auth ?
- champs sensibles ?

---

## 🔹 Étape 3 — tests

### 1. ID

```
GET /api/user?id=1 → id=2
```

---

### 2. Auth

```
Authorization: (remove)
Authorization: OTHER_TOKEN
```

---

### 3. Champs sensibles

```
{
  "role":"admin"
}
```

---

### 4. Format

```
Content-Type: application/x-www-form-urlencoded
```

---

### 5. Méthode

```
POST → PUT
```

---

## 🔹 Étape 4 — observer

- 200 / 403 / 500
- réponse
- différence

---

## 🔹 Étape 5 — creuser

👉 si changement → approfondir

---

# 🔹 10. Hypothèses + exemples

---

## 🔸 IDOR

```
GET /api/user?id=1
→ id=2
```

---

## 🔸 Mass Assignment

```
{
  "email":"a@test.com",
  "role":"admin"
}
```

---

## 🔸 Validation

```
{"id":"abc"}
{"id":-1}
{"id":[1]}
```

---

## 🔸 Business Logic

```
{"price":1}
{"discount":100}
```

---

## 🔸 Auth confusion

```
Authorization: USER_TOKEN
X-User-Id: 999
```

---

## 🔸 Parsing

```
JSON → form → XML
```

---

## 🔸 Méthode

```
GET → POST → PATCH → DELETE
```

---

# 🔹 11. Ce que tu dois éviter

❌ brute force  
❌ tester tout en même temps

---

# 🔹 12. Ce que tu dois faire

✔️ hypothèse  
✔️ test  
✔️ observation

---

# 🔹 13. Règle d’or

> 💥 Tu cherches ce à quoi le backend fait confiance

---

# 🔹 14. Mini checklist rapide

Quand tu vois une requête :

1. action ?
2. ID ?
3. auth ?
4. champs sensibles ?
5. format ?
6. méthode ?

---

# 🔥 Conclusion finale

> 💥 Le bug bounty = comprendre la logique backend  
> et trouver où elle fait confiance à de mauvaises données
