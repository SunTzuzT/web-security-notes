
## 1. Définition

La **path traversal** (ou **directory traversal**) est une vulnérabilité où une application laisse l’utilisateur **influencer un chemin de fichier** utilisé côté serveur.

Le serveur croit lire un fichier “prévu” :

/var/www/images/218.png

mais l’utilisateur réussit à lui faire lire autre chose :

/etc/passwd

---

## 2. Idée centrale

Le bug n’est **pas** `../` en soi.

Le vrai bug est :

- l’application fait confiance à une **entrée contrôlée par l’utilisateur**
- cette entrée influence un **chemin système**
- le backend valide parfois la **chaîne brute**
- mais le système de fichiers lit le **chemin final résolu**

## Phrase à retenir

> En path traversal, la vraie source de vérité n’est pas la chaîne reçue par l’application, mais le chemin final interprété par l’OS ou l’API filesystem.

---

# 3. Analogie simple

Imagine un **entrepôt**.

- Le serveur dit : “Tu peux demander un carton dans la zone **images**.”
- L’utilisateur donne un papier avec une direction.
- Le gardien lit seulement le début du papier et se dit : “Ça a l’air de parler de la zone images, donc OK.”
- Mais le cariste suit le chemin complet, qui contient des détours.

Résultat :

- le gardien croit autoriser un carton image
- le cariste finit dans la salle des archives RH

C’est exactement la path traversal :

- **check** = gardien
- **use** = cariste / système de fichiers

---

# 4. Exemple de base

## Cas normal

Frontend :

<img src="/loadImage?filename=218.png">

Backend logique :

base_dir = /var/www/images/  
path = base_dir + filename

Donc :

/var/www/images/218.png

## Cas vulnérable

L’utilisateur envoie :

../../../etc/passwd

Le chemin devient :

/var/www/images/../../../etc/passwd

Le système résout :

/etc/passwd

---

# 5. Pourquoi `../` marche

`../` veut dire :

- remonter d’un dossier
- sortir du répertoire prévu
- se déplacer vers un autre emplacement

Exemple :

/var/www/images/../../../etc/passwd

Lecture mentale :

- depuis `images`
- on remonte
- puis on cible `etc/passwd`

Sous Windows, on peut voir :

..\..\..\windows\win.ini

---

# 6. Impact

Un attaquant peut parfois lire :

- code source
- fichiers de config
- secrets
- identifiants backend
- clés API
- fichiers système

Et dans des cas plus graves, si l’écriture est possible :

- modifier des données
- modifier le comportement applicatif
- aller vers de la compromission serveur

---

# 7. Le pattern vulnérable

## Pattern classique

base_dir + user_input

Dangereux si :

- pas de validation forte
- pas de normalisation sûre
- pas de canonicalisation
- pas de vérification que le chemin final reste dans le dossier autorisé

---

# 8. Méthode générale de raisonnement

Quand tu rencontres un cas de path traversal, pose toujours cette question :

## Le backend valide quoi exactement ?

- la chaîne brute ?
- la chaîne après nettoyage ?
- la chaîne après décodage ?
- le chemin final résolu ?

## Workflow mental

input utilisateur  
-> décodage éventuel  
-> filtre éventuel  
-> normalisation éventuelle  
-> résolution du chemin  
-> lecture fichier

La faille existe si **le chemin final** sort toujours du dossier autorisé.

---

# 9. Cas pratique validé : lecture de `/etc/passwd`

## Observation typique

Requête :

GET /image?filename=../../../../etc/passwd

Réponse :

- `200 OK`
- contenu du body = `/etc/passwd`

Exemples vus :

root:x:0:0:...  
carlos:x:1200:12002:...  
administrator:x:...

## Ce que ça prouve

Ce n’est plus une hypothèse.

C’est une **lecture arbitraire de fichier confirmée**.

## Signal important

Si le serveur répond :

Content-Type: image/jpeg

mais le body contient du texte système, cela montre que :

- le backend ne valide pas une vraie image
- il renvoie juste **ce qu’il lit**

---

# 10. Tous les obstacles classiques et leur logique

---

## A. Traversal simple

### Défense absente

Payload :

../../../etc/passwd

### Pourquoi ça marche

Le backend concatène simplement le chemin.

### Leçon

Cas le plus direct : aucune vraie contrainte côté chemin final.

---

## B. Séquences `../` bloquées, mais chemin absolu accepté

### Situation

L’application bloque les traversals relatives :

- `../`
- `..\`

Mais accepte encore un chemin absolu.

### Payload

/etc/passwd

ou sous Windows :

C:\windows\win.ini

### Pourquoi ça marche

Le filtre protège seulement contre une **forme** (`../`), pas contre le fait qu’un utilisateur choisit encore librement une cible filesystem.

### Différence importante

- `etc/passwd` = relatif
- `/etc/passwd` = absolu

### Ce que tu as appris

Le slash initial change tout :

- sans slash → chemin relatif
- avec slash → chemin absolu

---

## C. Suppression non récursive des séquences de traversal

### Situation

L’application supprime `../`, mais une seule fois ou de manière naïve.

### Payload

....//....//....//etc/passwd

### Pourquoi ça marche

Le backend supprime une partie du motif, mais la chaîne restante se recompose en traversal valide.

Idée :

....//  ->  ../

après traitement partiel.

### Leçon

Le backend nettoie la chaîne, mais **ne revalide pas le résultat final**.

### Point clé

Ce n’est pas juste “une variante oubliée”.

C’est :

- nettoyage textuel fragile
- pas de contrôle sur le chemin résolu

---

## D. Nettoyage avant l’application : encodage / double encodage

### Situation

Le filtre ou le serveur web nettoie les traversals **avant** que l’application ne les utilise.

### Payloads possibles

Brut :

../../../etc/passwd

Encodé simple :

..%2f..%2f..%2fetc/passwd

Double encodé :

..%252f..%252f..%252fetc/passwd

### Pourquoi le double encodage peut marcher

Parce qu’il y a un décalage :

- le filtre regarde une représentation
- puis une autre couche décode plus tard
- la traversal n’apparaît qu’après

### Schéma

envoyé      : ..%252f..%252f..%252fetc/passwd  
vu au check : ..%252f..%252f..%252fetc/passwd  
après decode: ..%2f..%2f..%2fetc/passwd  
interprété  : ../../../etc/passwd

### Leçon

Tu ne cherches pas “un payload magique”.

Tu cherches :

> à faire apparaître la traversal **après** le moment du filtre.

---

## E. Mauvaise requête / mauvais paramètre

### Cas observé

Tu as testé sur :

GET /product?productId=...

et tu as reçu :

400 Invalid product ID

### Ce que ça veut dire

Tu n’étais pas sur le bon paramètre.

`productId` est un paramètre métier, pas un paramètre filesystem.

### Ce qu’il fallait viser

La requête image, par exemple :

GET /image?filename=...

### Leçon

Toujours identifier :

- quel paramètre pilote la logique métier
- quel paramètre pilote la lecture de fichier

---

## F. Validation du début du chemin (`startsWith` naïf)

### Situation

L’application exige que le chemin commence par :

/var/www/images

### Payload

/var/www/images/../../../etc/passwd

### Pourquoi ça marche

Le check voit :

- commence bien par `/var/www/images`

Mais le filesystem résout :

/etc/passwd

### Erreur backend

Le code fait l’équivalent de :

if path.startswith("/var/www/images"):  
    open(path)

### Faux raisonnement

“Commence par le bon dossier” ≠ “reste dans le bon dossier”

### Leçon

Un **prefix check textuel** n’est pas une preuve de confinement.

---

## G. Validation de l’extension avec null byte

### Situation

L’application veut que le nom se termine par `.png`.

### Payload

../../../etc/passwd%00.png

### Pourquoi ça marche

Après décodage :

../../../etc/passwd\0.png

Le check voit toute la chaîne et accepte `.png`.

Mais la couche vulnérable interprète `\0` comme une fin de chaîne, donc lit :

../../../etc/passwd

### Analogie

Étiquette :

passwd [STOP] .png

- le vigile lit tout et voit `.png`
- l’ouvrier s’arrête au panneau STOP
- il prend juste `passwd`

### Leçon

Encore le même problème :

- check sur représentation complète
- use sur représentation tronquée

### Nuance importante

Le null byte bypass est **historique / contextuel**.

Il dépend de :

- la stack
- le langage
- le runtime
- la façon dont la chaîne est convertie

Donc :

> `%00` n’est pas une vérité universelle, c’est une technique à tester selon contexte.

---

# 11. Pourquoi Intercept était parfois nécessaire

Tu as remarqué que pour certains labs, il fallait mettre **Intercept on**.

## Pourquoi

Ce n’est pas forcément une redirection.

Souvent :

- la page produit charge l’image automatiquement
- le navigateur envoie la requête ressource très vite
- sans interception, tu la vois dans l’historique mais tu ne la modifies pas à temps

## Donc

Intercept te permet de :

- capturer exactement la requête image
- modifier `filename`
- la laisser partir ensuite

## Leçon

Ne pas confondre :

- navigation principale
- chargement automatique d’une ressource secondaire

---

# 12. Comment raisonner proprement en Burp

## Étape 1 — Identifier le bon endpoint

Cherche la requête qui lit le fichier, par exemple :

GET /image?filename=21.jpg

Pas :

GET /product?productId=21

## Étape 2 — Comprendre le rôle exact du paramètre

Demande-toi :

- `filename` est-il un vrai nom de fichier ?
- un chemin complet ?
- un nom relatif ?
- une valeur ensuite décodée ?

## Étape 3 — Comparer les réponses

Toujours observer :

- code HTTP
- longueur
- message d’erreur
- contenu retourné

## Étape 4 — Ne pas envoyer du bruit

Tu testes :

1. brut
2. encodé simple
3. double encodé
4. absolu
5. obfusqué
6. null byte

Mais **un par un**, avec lecture des réponses.

---

# 13. Tableau mental des défenses naïves

|Défense naïve|Erreur réelle|Exemple de bypass|
|---|---|---|
|Bloquer `../`|Ne protège qu’un motif texte|`/etc/passwd`|
|Supprimer `../`|Nettoyage non récursif|`....//....//etc/passwd`|
|Filtrer avant décodage|Check avant transformation|`..%252f..%252f..%252fetc/passwd`|
|Vérifier `startsWith(base)`|Check textuel, pas canonique|`/var/www/images/../../../etc/passwd`|
|Imposer `.png`|Validation sur une chaîne, lecture sur une autre|`../../../etc/passwd%00.png`|

---

# 14. Le vrai pattern backend derrière tous les labs

Tous les labs enseignent la même chose :

## Pattern vulnérable

check sur forme A  
use sur forme B

Exemples :

- check sur la chaîne brute, use sur la chaîne décodée
- check sur le préfixe, use sur le chemin canonique
- check sur le suffixe, use sur la chaîne tronquée
- check après nettoyage partiel, use après reconstruction du chemin

## Phrase centrale

> Toutes les défenses cassées ici regardent la mauvaise représentation de l’entrée.

---

# 15. Connaissances complémentaires importantes

## A. Différence entre normalisation et canonicalisation

### Normalisation

Réduction syntaxique du chemin :

- simplifier `.` et `..`
- harmoniser les séparateurs

### Canonicalisation

Résolution plus complète du chemin final réel, parfois avec :

- `..`
- symlinks
- représentation absolue finale

En pratique, pour la sécurité, tu veux raisonner sur le **chemin canonique final**.

---

## B. Pourquoi le filesystem est la vraie source de vérité

L’application peut croire avoir construit :

/var/www/images/../../../etc/passwd

Mais ce n’est pas ce que “voit” réellement le système.

Le système, lui, résout :

/etc/passwd

Donc la sécurité doit se baser sur **ce chemin final**, pas sur la chaîne intermédiaire.

---

## C. Unix vs Windows

Unix :

../../../etc/passwd

Windows :

..\..\..\windows\win.ini

Selon la stack, il faut parfois tester :

- slash `/`
- backslash `\`
- versions encodées

---

## D. Tous les problèmes ne sont pas forcément exploitables

Un message d’erreur ou un rejet ne prouve pas la faille.

Exemple :

400 Invalid product ID

ça ne dit rien sur une traversal ; ça dit juste que tu n’as pas atteint la bonne logique.

Toujours raisonner depuis le **comportement observable réel**.

---

# 16. Prévention correcte

## Meilleure défense

Ne **jamais** laisser l’utilisateur fournir directement un chemin filesystem.

### Mieux

Le client envoie un identifiant logique :

imageId=218

Le serveur fait lui-même la correspondance :

218 -> /var/www/images/218.png

L’utilisateur ne contrôle plus le chemin.

---

## Si on doit quand même accepter une entrée liée à un fichier

Il faut **2 couches**.

### Couche 1 — Validation stricte d’entrée

Réduire la surface :

- whitelist
- seulement caractères attendus
- pas de slash
- pas de backslash
- pas de chemins absolus
- pas de `%00`

Exemple :

[a-zA-Z0-9._-]+

si on attend seulement un nom simple.

### Couche 2 — Vérification du chemin final

Processus correct :

1. joindre avec le base directory
2. canonicaliser / normaliser
3. vérifier que le chemin final reste dans la base autorisée

---

# 17. Code Java : explication claire

Exemple :

File file = new File(BASE_DIRECTORY, userInput);  
if (file.getCanonicalPath().startsWith(BASE_DIRECTORY)) {  
    // process file  
}

## Ce que fait ce code

### 1. Construire la cible

new File(BASE_DIRECTORY, userInput)

Ça prend le dossier de base + l’entrée.

### 2. Résoudre le vrai chemin

getCanonicalPath()

Cela résout :

- `../`
- représentations ambiguës
- parfois certains liens

### 3. Vérifier que la cible reste sous la base

Si le chemin final ne commence plus par la base, on rejette.

---

## Nuance importante

En pratique, mieux vaut aussi canonicaliser la base :

String base = new File(BASE_DIRECTORY).getCanonicalPath();  
String target = new File(BASE_DIRECTORY, userInput).getCanonicalPath();  
  
if (target.startsWith(base + File.separator)) {  
    // process file  
}

Sinon tu peux avoir des faux positifs texte comme :

/var/www/images_evil

qui commence textuellement par :

/var/www/images

alors que ce n’est pas le même dossier.

---

# 18. Ce qu’il faut retenir absolument

## Résumé ultra clair

### Le bug

L’utilisateur contrôle un chemin qui finit dans une API filesystem.

### Le vrai problème

Le backend valide souvent une représentation intermédiaire, pas le chemin final.

### Le bon réflexe offensif

Toujours demander :

- où a lieu le check ?
- où a lieu la transformation ?
- quelle valeur est réellement utilisée à la fin ?

### Le bon réflexe défensif

Toujours valider :

- une entrée restreinte
- puis le chemin canonique final

---

# 19. Synthèse finale

## Phrase maître

> La path traversal n’est pas une histoire de `../`. C’est une histoire de confiance mal placée dans un chemin utilisateur.

## Règle offensive

> Quand une défense regarde la mauvaise version de l’entrée, cherche à faire apparaître la traversal plus tard.

## Règle défensive

> La seule vérification robuste est celle du chemin final résolu, pas celle de la chaîne brute.

---

# 20. Mini fiche ultra compacte

## Path Traversal

**But** : lire ou écrire des fichiers arbitraires.

## Pattern vulnérable

base_dir + user_input

## Signaux

- fichier non image renvoyé par un endpoint image
- contenu de `/etc/passwd`
- incohérence entre `Content-Type` et body
- erreurs différentes selon représentation du chemin

## Bypass classiques

- `../../../etc/passwd`
- `/etc/passwd`
- `....//....//etc/passwd`
- `..%252f..%252f..%252fetc/passwd`
- `/var/www/images/../../../etc/passwd`
- `../../../etc/passwd%00.png`

## Vraie défense

- ne pas exposer le filesystem au client
- whitelist stricte
- canonicalisation
- vérifier que la cible finale reste dans le dossier autorisé

---

# 21. Conclusion pédagogique

Tous les labs que tu as faits enseignent une seule idée backend profonde :

> **entre ce que l’application croit lire et ce que le système lit vraiment, il peut y avoir un écart.**  
> C’est cet écart que l’attaquant exploite.

Et c’est pour ça que la path traversal est une excellente vulnérabilité d’apprentissage :

- elle t’oblige à penser en **source de vérité**
- en **check vs use**
- en **transformation de donnée**
- en **backend réel**, pas en payload magique
