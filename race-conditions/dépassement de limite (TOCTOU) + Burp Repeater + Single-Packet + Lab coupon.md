# 1) Idée centrale du cours

Une **race condition de dépassement de limite** apparaît quand une application impose une règle métier du type :

- “ce coupon n’est utilisable qu’une fois”
- “ce compte ne peut pas retirer plus que son solde”
- “ce CAPTCHA ne vaut qu’une fois”
- “cette action est limitée à une tentative”

…mais que le backend traite la logique ainsi :

CHECK  →  ACTION  →  WRITE

Le problème n’est pas la présence du check.  
Le problème est que le **check et l’écriture finale ne sont pas atomiques**.

Entre les deux, il existe une **petite fenêtre temporaire** où plusieurs requêtes peuvent voir le **même ancien état**, et donc toutes croire qu’elles ont le droit de passer.

---

# 2) Analogie ultra simple

## 🍪 L’armoire à biscuits

Imagine une maman qui dit :

> “Tu n’as droit qu’à 1 biscuit.”

Mais elle applique la règle comme ça :

1. elle regarde si tu as déjà pris un biscuit
2. si non, elle t’en donne un
3. ensuite seulement, elle écrit sur la feuille : “biscuit déjà pris”

Le bug est là :

Si **3 clones de toi** arrivent en même temps, la maman peut faire :

Clone 1 → "pas encore pris" → OK  
Clone 2 → "pas encore pris" → OK  
Clone 3 → "pas encore pris" → OK

Puis seulement après :

noter "déjà pris"

👉 Résultat : **1 règle, 3 biscuits**

C’est exactement ça une race condition de dépassement.

---

# 3) Le vrai mécanisme backend

## Workflow normal attendu

Exemple coupon :

1. vérifier si le coupon a déjà été utilisé  
2. appliquer la réduction  
3. mettre à jour la base pour marquer le coupon comme utilisé

## Le souci réel

Le backend passe par un **sous-état temporaire** :

"le coupon n’est pas encore marqué comme utilisé"

Ce sous-état commence quand la première requête démarre  
et se termine quand la DB est enfin mise à jour.

Pendant ce minuscule moment :

- Req1 lit : “coupon pas utilisé”
- Req2 lit : “coupon pas utilisé”
- Req3 lit : “coupon pas utilisé”

👉 toutes passent le check avant que l’état change

---

# 4) Nom exact de la faille : TOCTOU

TOCTOU = **Time Of Check To Time Of Use**

En français :

- **moment de la vérification**
- puis **moment de l’utilisation / écriture**

Le bug se situe dans l’écart entre les deux.

## Modèle mental

CHECK ---------[ race window ]--------- WRITE

Tant que plusieurs requêtes entrent dans cette zone avant le `WRITE`, elles peuvent toutes agir sur une vision périmée de l’état.

---

# 5) Exemples typiques

Cette logique ne concerne pas que les coupons.

## Exemples classiques

- appliquer un coupon plusieurs fois
- utiliser une carte cadeau plusieurs fois
- retirer plus que le solde
- transférer plus que le solde
- réutiliser un CAPTCHA censé être à usage unique
- bypass une limite anti-bruteforce
- soumettre plusieurs votes / évaluations

## Point commun

À chaque fois il existe une règle :

"tu n’as le droit qu’une seule fois / jusqu’à une certaine limite"

et un backend qui lit l’état puis l’écrit trop tard.

---

# 6) Ce qu’un hunter doit chercher

## Signaux backend utiles

- même requête envoyée plusieurs fois → résultats incohérents
- succès multiple alors qu’une seule action devrait être acceptée
- comportement très sensible au timing
- état final impossible selon la logique métier normale
- réponses qui ne reflètent pas forcément le succès réel, alors que l’état final a changé

## Question hunter à se poser

Quelle est la ressource limitée ?  
À quel moment le backend décide qu’elle est encore disponible ?  
À quel moment il écrit qu’elle ne l’est plus ?

---

# 7) Burp Repeater : pourquoi ce n’est pas “envoyer vite”

Le but n’est pas de “spammer”.

Le but est de **synchroniser** les requêtes pour qu’elles atteignent le backend dans la **même race window**.

## Mauvaise vision

"je vais juste envoyer beaucoup de requêtes"

## Bonne vision

"je vais faire arriver plusieurs requêtes au même moment backend"

---

# 8) Le vrai ennemi : le jitter

## Définition

Le **jitter** = variation imprévisible du délai.

Deux requêtes identiques, envoyées “en même temps”, peuvent en réalité arriver à des moments différents à cause de :

- réseau
- congestion
- ordonnanceur OS
- queue serveur
- latence interne

## Pourquoi c’est grave

Si Req1 arrive un peu avant Req2 :

Req1 → check OK  
Req1 → write DB  
Req2 → arrive après → check FAIL

👉 la collision n’a pas lieu

## Résumé

Le jitter désynchronise.  
La race a besoin d’alignement.

---

# 9) Comment Burp aide : Repeater 2023.9+

Burp Repeater moderne aide à réduire l’impact du jitter.

## Selon le protocole

### HTTP/1 → last-byte sync

Burp prépare plusieurs requêtes et libère leur dernier octet en même temps.

### HTTP/2 → single-packet attack

Burp pousse plusieurs requêtes dans un seul envoi logique TCP pour qu’elles arrivent quasiment ensemble.

👉 L’objectif dans les deux cas :  
**faire converger plusieurs requêtes vers le même instant backend**

---

# 10) Single-packet : explication simple

Le nom “single-packet” prête à confusion.

Il ne faut pas comprendre :

1 requête = 1 paquet  
ou  
30 requêtes = 1 énorme paquet magique

Ce qu’il faut comprendre :

- plusieurs requêtes HTTP sont construites
- elles sont mises dans le même **flux TCP**
- TCP les envoie **sans pause**
- le serveur les reçoit comme un bloc continu
- elles arrivent au parser HTTP quasi ensemble

👉 le gain n’est pas “la taille”,  
👉 le gain est **le timing**

---

# 11) Single-packet avec le modèle OSI

---

## 🧱 L7 — Application

C’est le niveau HTTP.

Burp ou ton navigateur construit plusieurs requêtes :

POST /cart/coupon  
POST /cart/coupon  
POST /cart/coupon  
...

À ce niveau, ce sont juste des **données applicatives**.

HTTP n’est pas un paquet réseau.  
HTTP est un **flux de données**.

---

## 🚚 L4 — Transport (TCP)

Le système d’exploitation prend ces données et les met dans un **buffer TCP**.

### Buffer TCP

Le buffer = une **zone mémoire temporaire** qui stocke les données avant envoi ou après réception.

### Ce que fait TCP

- il reçoit le flux HTTP
- il le stocke temporairement dans le buffer
- il le **segmente** en morceaux adaptés au transport réseau

### Segmentation

TCP ne peut pas forcément envoyer tout d’un seul bloc physique.  
Il découpe selon la taille autorisée (MSS).

Exemple :

Segment 1 → Req1 + début Req2  
Segment 2 → fin Req2 + Req3  
Segment 3 → Req4 + Req5

👉 important :  
ce ne sont pas plusieurs envois espacés dans le temps  
mais un **flux continu envoyé sans pause**

---

## 🌍 L3 — Réseau (IP)

Chaque segment TCP est encapsulé dans un paquet IP.

Le rôle d’IP :

- adresser
- router
- livrer les paquets à destination

À ce niveau, il n’y a rien de “métier”.  
C’est juste le transport réseau.

---

## 🖥️ Côté serveur

Le serveur reçoit les paquets IP, puis :

### L4 TCP serveur

- récupère les segments
- les remet dans l’ordre
- reconstruit le flux original dans son propre buffer

### L7 HTTP parser

- lit ce flux
- en extrait les requêtes HTTP individuelles
- les transmet à la logique applicative/backend

👉 le point clé :  
**plusieurs requêtes sont déjà présentes ensemble dans la mémoire serveur**

Donc le backend peut les traiter presque en parallèle.

---

# 12) Schéma global complet

==================== CLIENT (ton PC / Burp) ====================  
  
[L7 - HTTP]  
Req1  
Req2  
Req3  
Req4  
 ↓  
flux HTTP concaténé  
  
---------------------------------------------------------------  
  
[L4 - TCP]  
Buffer TCP client :  
"Req1Req2Req3Req4"  
  
Segmentation :  
Segment 1 → Req1 + début Req2  
Segment 2 → fin Req2 + Req3  
Segment 3 → Req4  
  
⚠️ envoyés sans pause  
  
---------------------------------------------------------------  
  
[L3 - IP]  
Paquet IP 1 → Segment 1  
Paquet IP 2 → Segment 2  
Paquet IP 3 → Segment 3  
  
==================== RÉSEAU ====================  
  
Transport des paquets  
→ très rapprochés  
→ peu de jitter entre eux  
  
==================== SERVEUR ====================  
  
[L3 - IP]  
réception des paquets  
  
---------------------------------------------------------------  
  
[L4 - TCP]  
Buffer TCP serveur :  
réassemblage du flux  
  
"Req1Req2Req3Req4"  
  
---------------------------------------------------------------  
  
[L7 - HTTP parser]  
Découpe :  
Req1  
Req2  
Req3  
Req4  
  
⚠️ toutes déjà là en mémoire  
  
==================== BACKEND ====================  
  
Req1 → check  
Req2 → check  
Req3 → check  
  
💥 race window  
  
Req1 → write  
Req2 → write  
Req3 → write

---

# 13) Où se joue réellement la faille

Pas dans IP.  
Pas dans TCP.

La faille se joue ici :

HTTP parser → backend logic → DB/cache/session

Plus précisément :

lecture état actuel → décision → écriture trop tardive

Le réseau n’est qu’un **levier de synchronisation**.  
La vulnérabilité, elle, est **métier/backend**.

---

# 14) Workflow d’exploitation générique avec Repeater

## Étape 1 — Identifier un endpoint à ressource limitée

Exemples :

- coupon
- checkout
- gift card
- transfer
- password reset
- verify code
- rate limit

## Étape 2 — Faire un baseline

Envoyer la requête une fois et observer le comportement normal.

## Étape 3 — Chercher la contrainte

Exemple :

- première fois → `Coupon applied`
- deuxième fois → `Coupon already applied`

👉 tu as identifié la limite

## Étape 4 — Grouper et dupliquer

- même requête
- même session
- même host
- même protocole
- même target

## Étape 5 — Envoi parallèle

- `Send group in parallel`
- ou `single connection` si requis

## Étape 6 — Vérifier l’état final

Ne jamais te fier uniquement aux réponses HTTP.  
Toujours vérifier :

- panier
- commandes
- solde
- historique
- effet final backend

---

# 15) Piège Burp important : homogénéité obligatoire

Pour un envoi propre en single connection / single packet, Burp exige souvent :

- même host exact
- même port
- même schéma
- même protocole HTTP
- même target

## Exemple de bug classique

Le `Host:` dans la requête dit :

Host: 0a3900f8033a22fc81f7a7d7009000c2.web-security-academy.net

Mais la target Burp est mal configurée avec un host tronqué.

👉 Burp refuse car il ne voit pas les requêtes comme allant vers la **même connexion TCP logique**.

---

# 16) Pourquoi on ne voit pas forcément les single-packet dans Proxy

Parce que :

- **Proxy** montre surtout le trafic du client configuré via le proxy
- **Repeater** est un outil interne Burp qui émet lui-même les requêtes
- en single-packet, plusieurs requêtes HTTP partent dans le même flux TCP

Donc le meilleur endroit pour observer :

- les réponses de groupe dans Repeater
- le Logger
- l’état final backend

Pas nécessairement Proxy history.

---

# 17) Le lab : vision d’ensemble

## But

Acheter une **L33t Leather Jacket (1337$)** avec seulement **50$**.

## Première intuition

On pourrait croire que la faille est sur `/cart/checkout`.

## Réalité du lab

La race exploitable est plus tôt, sur :

POST /cart/coupon

Le backend permet d’appliquer **plusieurs fois** un coupon censé être unique.

👉 Le checkout ne fait ensuite que **consommer un panier déjà corrompu**

---

# 18) Vérifications critiques à faire avant d’exploiter

Ces vérifications sont très importantes, car elles te disent **où est la source de vérité**.

---

## ✅ Vérification 1 — Où est stocké le panier ?

Prends :

GET /cart

Puis envoie-la :

- **avec le cookie de session**
- **sans le cookie de session**

## Résultat attendu

- avec cookie → panier rempli
- sans cookie → panier vide

## Conclusion

Le panier est stocké **côté serveur**, identifié par la session.

👉 donc :

- l’état n’est pas dans le client
- la vraie source de vérité est backend
- cela crée un vrai risque de collision sur cet état centralisé

---

## ✅ Vérification 2 — Quelle est la restriction ?

Teste deux fois :

POST /cart/coupon  
coupon=PROMO20

## Résultat attendu

- première fois → `Coupon applied`
- deuxième fois → `Coupon already applied`

## Conclusion

Le backend impose bien la règle :

ce coupon ne doit être appliqué qu’une seule fois

---

## ✅ Vérification 3 — Où la décision semble-t-elle prise ?

Si le body de `/cart/checkout` est quasi vide et ne contient ni prix ni produit ni coupon, cela indique souvent que :

- l’état du panier est déjà stocké côté serveur
- la requête finale lit cet état serveur
- le client ne fait que déclencher une décision backend

---

# 19) Pourquoi on s’est d’abord trompé sur `/checkout`

C’était une hypothèse logique :

- `/coupon` prépare l’état
- `/checkout` consomme l’état

Et souvent, dans beaucoup de vraies applis, la race utile est bien sur la consommation finale.

Mais dans **ce lab précis**, la ressource limitée n’est pas :

"le droit de payer"

La ressource limitée est :

"le droit d’appliquer PROMO20 une seule fois"

👉 donc le vrai endpoint vulnérable est `/cart/coupon`

Très bonne leçon :  
**la ressource limitée réelle n’est pas toujours celle qu’on croit au début.**

---

# 20) Analogie du lab

## 🎟️ Le ticket de réduction

Imagine un vendeur qui dit :

> “Ce ticket promo ne peut être utilisé qu’une fois.”

Son workflow :

1. il regarde si le ticket est déjà tamponné
2. s’il ne l’est pas, il applique la remise
3. ensuite seulement, il tamponne “utilisé”

Toi, tu envoies 20 clones à la caisse.

Le vendeur fait :

Clone1 → pas tamponné → OK  
Clone2 → pas tamponné → OK  
Clone3 → pas tamponné → OK  
...

Puis seulement après :

tamponner = utilisé

👉 Résultat :  
le même ticket est accepté plusieurs fois  
la remise s’accumule  
le panier devient absurdement peu cher

---

# 21) Workflow exact du lab

## Phase A — Préparation

- login : `wiener:peter`
- acheter un petit article pour observer le process
- identifier :
    - `POST /cart`
    - `POST /cart/coupon`
    - `POST /cart/checkout`

## Phase B — Vérifs

- retester `/cart` avec/sans cookie
- confirmer que le panier est lié à la session
- confirmer que le coupon est normalement unique

## Phase C — Baseline

Envoyer plusieurs `POST /cart/coupon` **en séquence**.

Résultat attendu :

- 1 succès
- puis uniquement `Coupon already applied`

👉 comportement normal

## Phase D — Race

- retirer le coupon
- grouper 20 à 30 requêtes `POST /cart/coupon`
- envoyer en parallèle

Résultat attendu :

- plusieurs réponses `Coupon applied`

## Phase E — Vérification état final

Rafraîchir `/cart`

Tu dois voir la réduction appliquée plusieurs fois.

## Phase F — Exploit complet

- mettre la **L33t jacket** dans le panier
- rejouer la race coupon
- recommencer si nécessaire
- vérifier que :

TOTAL < 50$

- puis checkout normal

👉 lab résolu

---

# 22) Pourquoi les réponses seules peuvent tromper

En race condition, les réponses ne racontent pas toujours toute la vérité.

Tu peux avoir :

- des réponses toutes similaires
- voire des réponses d’échec
- mais un état final qui a réellement changé

## Règle d’or

Toujours corréler :

- réponses HTTP
- état final backend
- effet métier réel

Dans ce lab, le vrai signal final est :

le total du panier a chuté de manière impossible

---

# 23) Le piège “je duplique une requête qui échoue ?”

Oui, parfois une requête qui **échoue seule** est justement la bonne requête à dupliquer.

Pourquoi ?

Parce qu’en race condition, tu ne cherches pas une requête magique différente.  
Tu cherches à faire exécuter **la même décision** plusieurs fois sur un état non encore mis à jour.

Exemple générique :

Req1 seule → FAIL  
Req2 seule → FAIL

Mais :

Req1 + Req2 ensemble → lisent un état intermédiaire → collision possible

Dans ce lab précis, la collision utile s’est révélée être sur `/cart/coupon`, pas sur `/checkout`.

---

# 24) Modèle mental ultra important : PREPARE vs CONSUME

Dans une app métier, pense souvent en deux temps :

PREPARE STATE   →   CONSUME STATE

Exemples :

- coupon → checkout
- reset token creation → password reset confirm
- reservation lock → payment confirm
- draft → publish
- claim voucher → redeem balance

## Leçon

La race peut être :

- sur la **préparation** de l’état
- ou sur la **consommation** de l’état

Dans le lab :

- préparation = `/cart/coupon`
- consommation = `/cart/checkout`

La faille exploitable était sur la **préparation**.

---

# 25) Comment raisonner proprement sur une race

## Checklist hunter

### A. Identifier la ressource limitée

- coupon unique ?
- crédit ?
- solde ?
- quota ?
- token one-time ?

### B. Identifier la source de vérité

- session serveur ?
- DB ?
- cache ?
- microservice ?

### C. Trouver le check

- “already applied”
- “insufficient funds”
- “too many attempts”
- “already used”

### D. Trouver le write

- flag en DB
- décrément de solde
- ajout de réduction
- création de commande
- mise à jour compteur

### E. Poser la question clé

Puis-je exécuter plusieurs fois le check avant le write ?

Si oui → race possible

---

# 26) Pourquoi l’état côté serveur est si important

Quand l’état est centralisé côté serveur :

- plusieurs requêtes concurrentes lisent la **même source de vérité**
- si le backend ne verrouille pas correctement
- alors toutes peuvent voir le même ancien état

Dans le lab, le test avec/sans cookie prouve justement ça :

- cookie = accès à la session qui contient le panier
- pas de cookie = pas de panier

👉 Donc la session côté serveur est le support de l’état métier

---

# 27) Le vrai rôle du buffer TCP

Le buffer TCP est une **boîte mémoire temporaire**.

## Côté client

Il sert à :

- accumuler les données HTTP
- préparer l’envoi
- les pousser en bloc

## Côté serveur

Il sert à :

- recevoir les segments
- les remettre ensemble
- fournir un flux cohérent au parser HTTP

👉 Dans le single-packet, le buffer aide à ce que **plusieurs requêtes soient déjà présentes ensemble** quand le serveur commence à les traiter.

---

# 28) Le vrai rôle de la segmentation TCP

La segmentation = découpage du flux en morceaux transportables.

Ce n’est pas une “fonction sécurité”, ni une “fonction métier”.

C’est juste une adaptation au réseau.

## À retenir

Le fait que TCP segmente ne casse pas la synchro si :

- tous les segments sont envoyés **sans pause**
- dans la même connexion
- dans le même mouvement logique

👉 encore une fois :  
la magie n’est pas dans la taille  
elle est dans **l’absence de délai entre les segments**

---

# 29) Pourquoi Turbo Intruder existe

Repeater suffit souvent pour les labs simples.

Mais certaines races sont plus dures :

- fenêtre minuscule
- besoin de centaines/milliers de tentatives
- jitter interne serveur important
- besoin d’automatisation

Dans ce cas, Turbo Intruder apporte :

- gros volume
- répétition
- orchestration fine
- meilleur contrôle de la concurrence

## Analogie

- Repeater = tu lances une poignée de fléchettes en même temps
- Turbo Intruder = tu installes une machine qui les lance en rafale jusqu’à toucher

---

# 30) Correctifs côté dev

Une vraie correction ne consiste pas à “rajouter un message d’erreur”.

Il faut rendre l’opération **atomique**.

## Correctifs robustes

- transaction DB
- verrou (`SELECT ... FOR UPDATE`)
- lock applicatif
- décrément/check atomique
- idempotency key
- mise à jour de l’état avant ou pendant l’action, pas après
- éviter lecture puis écriture séparées sans protection

## Mauvais correctifs

- messages UI
- délais artificiels
- vérifs purement front
- logs sans blocage réel

---

# 31) Résumé premium final

## En une phrase

Une race condition de dépassement arrive quand le backend fait :

je vérifie maintenant  
mais je n’écris la vérité qu’après

et que plusieurs requêtes profitent de cet intervalle.

## Dans le lab

- la ressource limitée = **le droit d’appliquer PROMO20 une fois**
- la source de vérité = **panier/session côté serveur**
- le bug = **plusieurs `/cart/coupon` passent avant que l’état “coupon déjà appliqué” soit écrit**
- l’effet = **réductions cumulées**
- l’exploit final = **L33t jacket < 50$ → checkout normal**

---

# 32) Raccourcis mentaux à retenir

## Raccourci 1

Race condition = problème de timing backend, pas de payload

## Raccourci 2

HTTP = logique  
TCP = timing  
Backend = vulnérabilité

## Raccourci 3

Ce n’est pas “envoyer vite”  
C’est “faire lire le même ancien état à plusieurs requêtes”

## Raccourci 4

Toujours chercher :  
CHECK → RACE WINDOW → WRITE

## Raccourci 5

PREPARE STATE / CONSUME STATE  
La race peut être dans l’un ou dans l’autre

---

# 33) Mini fiche hunter réutilisable

## Quand tu vois une limite métier, demande-toi :

- quelle est la ressource rare ?
- où est stocké son état ?
- qui la lit ?
- qui la met à jour ?
- quand la mise à jour devient visible ?
- plusieurs requêtes peuvent-elles lire avant écriture ?

## Signaux positifs

- réponses incohérentes
- succès multiples
- état final impossible
- réduction/solde/commande anormale
- comportement sensible à la synchro

## Méthode

- baseline
- vérif source de vérité
- vérif restriction
- groupe de requêtes
- envoi parallèle
- vérif état final

---

# 34) Phrase finale à garder en tête

Si plusieurs requêtes peuvent lire un état non encore mis à jour,  
alors la logique métier peut être cassée.

Si tu veux, je peux maintenant te transformer cette fiche en **version Obsidian ultra propre**, avec titres courts, callouts, emojis homogènes et blocs markdown prêts à coller.
