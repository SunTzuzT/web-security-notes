# 🎯 Idée principale

Une **race condition** n’est pas “le site est rapide” ou “j’envoie beaucoup de requêtes”.

Le vrai problème, c’est :

> **le serveur fait plusieurs petites étapes en secret, et pendant ce temps-là, son état peut être temporairement faux ou incomplet.**

Et si toi tu en profites **au bon moment**, tu peux lui faire faire une bêtise.

---

# 🧸 Analogie ultra simple : le maître d’école

Imagine un maître d’école.

Un enfant lui dit :

> “Je veux sortir en récréation.”

Le maître fait ça :

1. il écrit ton nom sur la liste des enfants autorisés
2. il doit encore vérifier si tes parents ont signé le papier
3. ensuite seulement il dit oui ou non

Le problème ?

👉 **entre 1 et 3**, ton nom est **déjà sur la liste**,  
mais la vérification n’est **pas encore finie**.

Donc si à ce moment-là tu cours vers la porte et que le surveillant regarde juste la liste, il voit ton nom et dit :

> “Ah, il est autorisé, vas-y.”

Alors qu’en vrai, le maître n’avait **pas encore terminé**.

---

# 💡 Traduction technique

- le **maître** = le backend
- la **liste** = la session / DB / cache
- la **vérification des parents** = le contrôle de sécurité
- le **surveillant** = un autre endpoint
- **courir vers la porte au bon moment** = envoyer une autre requête en parallèle

---

# ⚠️ Le point le plus important

Une seule requête HTTP peut déclencher **plusieurs étapes backend invisibles**.

Exemple :

POST /login  
    ↓  
1. session['userid'] = user.id  
2. session['enforce_mfa'] = true  
3. redirect /mfa

Le danger est ici :

1. session['userid'] = user.id   ← déjà connecté un peu  
2. MFA pas encore forcé          ← pas encore totalement protégé

Donc pendant un tout petit instant :

> **tu as une session valide, mais le MFA n’est pas encore vraiment actif**

C’est ça un **sous-état**.

---

# 🧩 C’est quoi un “sous-état” ?

Un **sous-état**, c’est :

> **un état temporaire du backend pendant qu’il est encore en train de travailler**

Il n’a pas fini son traitement, mais il a déjà commencé à modifier des choses.

Exemples :

- session créée, mais pas encore MFA
- coupon accepté, mais pas encore marqué “utilisé”
- crédit vérifié, mais commande pas encore confirmée
- token généré, mais pas encore lié proprement

---

# 🔐 Atomique vs non atomique

## ✅ Atomique

Une action est **atomique** si elle est faite en **un seul bloc**.

En gros :

> soit tout est fait  
> soit rien n’est fait  
> mais il n’y a **pas d’état intermédiaire visible**

### Exemple safe

vérifier + écrire = une seule opération

---

## ❌ Non atomique

Une action est **non atomique** si elle est faite en plusieurs morceaux.

### Exemple vulnérable

1. vérifier  
2. traiter  
3. écrire

Là, entre 1 et 3 :

> il existe un moment exploitable

---

# 🏎️ Séquentiel vs parallèle

## Séquentiel

Tu envoies :

Req1 → fin  
Req2 → fin

Le serveur a le temps de finir la première avant la deuxième.

---

## Parallèle

Tu envoies :

Req1  
Req2

presque en même temps.

Le serveur peut alors :

- lire un état pas encore mis à jour
- traiter deux actions sur la même donnée
- croire deux choses différentes en même temps

---

# 🔑 Clé d’écriture

C’est un concept très important.

La **clé d’écriture**, c’est :

> **la chose que le backend utilise pour savoir quel état il doit modifier**

Exemples :

- `userid`
- `sessionid`
- `cartId`
- `token`
- `orderId`

### Exemple

UPDATE carts SET total = ... WHERE cart_id = 123

Ici la clé d’écriture = `cart_id`

---

# Pourquoi c’est crucial ?

Parce qu’une collision n’existe que si **deux requêtes touchent la même chose**.

### Pas de collision

- requête A modifie le panier de Alice
- requête B modifie le panier de Bob

➡️ pas la même clé, donc peu de risque

### Collision possible

- requête A modifie `cartId=123`
- requête B modifie aussi `cartId=123`

➡️ là, oui, ça peut se rentrer dedans

---

# 🧠 Mentalité correcte

Ne pense pas :

> “où est la race condition ?”

Pense plutôt :

> **quel état backend devient vrai trop tôt ?**

ou

> **quel état temporaire existe avant que le serveur ait fini tous ses contrôles ?**

C’est ça la vraie question.

---

# 🔍 La méthode : Predict → Probe → Prove

---

## 1. PREDICT = prédire

Tu cherches **où** ça pourrait casser.

Tu te poses ces questions :

- est-ce un flow critique ?
- est-ce qu’il y a plusieurs étapes backend ?
- est-ce que quelque chose est écrit avant la fin ?
- est-ce que deux requêtes peuvent toucher la même donnée ?

### Flows typiques

- login + MFA
- reset password
- coupon
- panier / checkout
- changement d’email
- wallet / crédit

---

## 2. PROBE = sonder

Tu compares :

- **séquentiel**
- **parallèle**

Tu gardes les mêmes requêtes.  
Tu changes seulement le **timing**.

Tu regardes si quelque chose change :

- un 200 inattendu
- un 403 qui disparaît
- un token différent
- un panier incohérent
- un crédit mal décrémenté
- un accès autorisé trop tôt

---

## 3. PROVE = prouver

Une fois que tu vois un signal :

- tu réduis au minimum
- tu gardes seulement les requêtes utiles
- tu reproduis plusieurs fois
- tu comprends **quelle primitive exacte** tu as trouvée
- tu montres l’impact

---

# 🧬 Signal, primitive, impact

C’est très important de distinguer les 3.

## Signal

C’est le petit indice.

Exemples :

- une réponse différente
- un 200 bizarre
- un comportement instable

## Primitive

C’est **ce que le bug te donne réellement**.

Exemples :

- double usage d’un coupon
- session valide avant MFA
- panier modifiable après validation
- token mélangé

## Impact

C’est ce que tu peux faire grâce à ça.

Exemples :

- bypass MFA
- achat gratuit
- account takeover
- abus financier

---

# 🔥 Exemple 1 — Login + MFA

## Flow normal

POST /login  
→ créer session  
→ activer MFA  
→ rediriger vers /mfa

## Bug possible

Le backend fait :

1. session['userid'] = X  
2. session['enforce_mfa'] = true

Donc pendant un court instant :

- tu as déjà `userid`
- mais MFA n’est pas encore forcé

## Exploit

Tu envoies en parallèle :

- `POST /login`
- `GET /my-account`

Si `/my-account` passe parfois sans MFA :

✅ race confirmée

## Primitive

> session partiellement authentifiée exploitable

## Impact

> bypass MFA

---

# 💸 Exemple 2 — Coupon one-time

## Flow normal

1. vérifier si coupon déjà utilisé  
2. appliquer remise  
3. marquer coupon utilisé

## Bug possible

Deux requêtes parallèles lisent toutes les deux :

used = false

avant que l’une écrive :

used = true

## Résultat

Les deux coupons passent.

## Primitive

> double consommation d’une ressource one-time

## Impact

> réduction multiple / achat gratuit

---

# 🛒 Exemple 3 — Multi-endpoint race sur un panier

C’est le plus important pour les race conditions multiples.

## Analogie simple : le restaurant

Tu commandes une salade à 10€.

Le serveur vérifie :

> “tu as bien 10€ ?”

Pendant qu’il va en cuisine, tu cries au cuisinier :

> “Finalement mets-moi le gros steak à 1400€”

Le serveur revient, pense avoir validé la salade,  
mais la cuisine sert le steak.

---

# Traduction backend

- `POST /cart` = modifier le panier
- `POST /cart/checkout` = lancer le paiement / confirmation
- panier stocké côté serveur = même état partagé

## Le vrai bug

Le serveur :

1. vérifie si ton crédit suffit pour le panier actuel
2. puis confirme la commande un peu plus tard

Mais entre les deux :

👉 une autre requête peut modifier le panier

Donc :

> **le paiement est validé sur un état, mais la commande finale utilise un autre état**

---

# 🧠 Ce qu’il faut retenir de ce lab

Ce n’est pas :

> “j’ai changé le productId”

Le vrai mécanisme est :

> **le backend suppose que le panier ne change pas entre la validation du paiement et la confirmation finale**

Et cette hypothèse est fausse.

---

# 🔥 Warm-up de connexion

Parfois, ton premier appel est lent juste parce que la connexion backend n’est pas encore prête.

Donc tu ajoutes une requête inutile avant, par exemple :

GET /

Elle ne crée pas la faille.  
Elle sert juste à :

> **préparer la connexion pour que les vraies requêtes critiques arrivent au bon moment**

---

# ✅ Comment reconnaître une race intéressante

Pose-toi toujours ces 5 questions :

1. **Quelle est la source de vérité ?**
    - session ?
    - DB ?
    - cache ?
    - wallet ?
    - panier ?
2. **Quelle est la clé d’écriture ?**
    - userId ?
    - sessionId ?
    - cartId ?
    - token ?
3. **Quel état est écrit trop tôt ?**
    - session créée ?
    - coupon pas encore “used” ?
    - paiement validé ?
    - token encore valide ?
4. **Quel endpoint peut profiter de cet état intermédiaire ?**
5. **Le parallèle donne-t-il un résultat impossible en séquentiel ?**

---

# 🧪 Comment tester concrètement

## Baseline

Toujours faire d’abord :

- séquentiel
- noter le comportement normal

## Puis parallèle

Même requêtes.  
Même paramètres.  
Juste timing différent.

## Ce que tu cherches

- 2 succès alors qu’1 seul devrait passer
- un accès obtenu trop tôt
- un état final incohérent
- une différence visible uniquement en parallèle

---

# ⚠️ Erreur classique

Ne cherche pas :

> “une grosse différence spectaculaire”

Cherche :

> **la plus petite incohérence reproductible**

C’est souvent ça le vrai signal.

---

# 📌 Résumé ultra simple

## Une race condition, c’est :

> le serveur est au milieu de son travail  
> et toi tu lui fais faire autre chose pendant qu’il n’a pas fini

## Un sous-état, c’est :

> un état temporaire caché pendant le traitement

## Atomique, c’est :

> pas d’état intermédiaire exploitable

## Clé d’écriture, c’est :

> la chose qui dit au backend quel état il modifie

## Multi-endpoint race, c’est :

> deux endpoints différents touchent le même état au même moment

## Predict / Probe / Prove, c’est :

- **Predict** : où ça peut casser ?
- **Probe** : est-ce que le parallèle change quelque chose ?
- **Prove** : quel bug exact j’ai, et quel impact réel ?

---

# 🧠 Phrase mentale finale à retenir

> **Je ne cherche pas une requête magique.  
> Je cherche un moment où le backend croit quelque chose… alors que ce n’est déjà plus vrai.**

---

Si tu veux, je peux te faire juste après une **version encore plus premium “prête à coller dans Obsidian”**, avec :

- emojis propres
- schémas ASCII
- section “leçon de hunt”
- section “signaux Burp à surveiller”

c quoi shrema asci

Tu veux dire **“schéma ASCII”** (pas _shrema_ 🙂).

---

# 🧠 Définition simple

Un **schéma ASCII**, c’est :

> 👉 un dessin fait uniquement avec du texte (clavier)

Donc **pas d’image**, juste des caractères comme :

|  -  >  ↓  →  [ ]

---

# 📦 Exemple basique

## Sans schéma

> login → session → MFA → accès

## Avec schéma ASCII

POST /login  
     ↓  
[ session créée ]  
     ↓  
[ MFA activé ]  
     ↓  
[ accès autorisé ]

---

# 💣 Exemple utile (race condition)

Req A: POST /login  
        ↓  
    [ session.userid = X ]   ← ⚠️ état temporaire  
        ↓  
    [ MFA pas encore actif ]  
  
Req B: GET /my-account  
        ↓  
    accès possible ❗

---

# 🎯 Pourquoi on utilise ça

Parce que ça permet de :

- visualiser les étapes backend
- voir **où est la faille**
- comprendre le timing

👉 surtout en bug bounty, c’est très utile pour :

- ton cerveau
- tes notes Obsidian
- tes reports

---

# 🧠 Résumé

> ASCII = dessin avec du texte  
> schéma ASCII = expliquer un flow sans image

---

Si tu veux, je peux te faire :  
👉 un **schéma ASCII parfait du lab panier (checkout race)**  
qui va te faire tout comprendre en 10 secondes.

oui

# 🧠 Schéma ASCII — Race condition panier (ultra clair)

---

## 🎯 Situation normale (SAFE)

Panier = [ Gift card $10 ]  
  
        POST /cart/checkout  
                ↓  
        [ Lire total = $10 ]  
                ↓  
        [ Vérifier crédit OK ]  
                ↓  
        [ Confirmer commande ]  
                ↓  
        Résultat = tu reçois gift card

👉 Tout est cohérent  
👉 Ce que tu paies = ce que tu reçois

---

## 💣 Version vulnérable (RACE)

État initial:  
Panier = [ Gift card $10 ]  
  
  
Req A: POST /cart/checkout  
                ↓  
        [ Lire total = $10 ]  
                ↓  
        ⚠️ FENÊTRE CRITIQUE  
  
  
Req B: POST /cart (modifié)  
        productId = veste ($1400)  
                ↓  
        [ Panier devient = veste ]  
  
  
Req A continue  
                ↓  
        [ Vérification OK (basée sur $10) ]  
                ↓  
        [ Confirmation commande ]  
                ↓  
        Résultat = tu reçois la veste 😈

---

# 🔥 Le point EXACT où ça casse

[ Lire total = $10 ]  
        ↓  
⚠️ le panier change ici  
        ↓  
[ Confirmer commande = veste ]

👉 Le serveur :

- **vérifie un état (gift card)**
- **confirme un autre état (veste)**

---

# 🧠 Traduction simple

> Le serveur regarde une chose…  
> mais agit sur une autre

---

# ⚙️ Version backend (réaliste)

1. total = read(cart)  
2. if total <= credit:  
3.     process_payment()  
4. confirm_order(cart)

💥 Problème :

- `read(cart)` → ancien panier
- `confirm_order(cart)` → nouveau panier

---

# 🔑 Pourquoi ça marche

Parce que :

- panier stocké côté serveur (session / cartId)
- `/cart` et `/checkout` modifient **la même clé**
- pas de verrou entre validation et confirmation

---

# ⚠️ Rôle du timing

T1: checkout lit panier  
T2: cart modifie panier  
T3: checkout confirme panier modifié

👉 Tu exploites **T2 entre T1 et T3**

---

# ⚡ Version ultra simplifiée

Serveur:  
"Tu payes 10€ ? OK"  
  
Toi:  
"tiens maintenant c’est 1400€"  
  
Serveur:  
"ok validé"

---

# 🧠 Résumé mental final

- 🔑 clé = panier (cartId / session)
- ⚠️ faille = pas atomique
- 🧨 primitive = désynchronisation check vs confirm
- 💥 impact = payer moins que ce que tu reçois

---

# 🎯 Phrase à retenir (très importante)

> **Le backend valide une chose…  
> mais exécute avec une autre.**

---

Si tu veux, prochaine étape :  
👉 je te fais le **même schéma ASCII pour MFA bypass** (encore plus important en bug bounty réel).


# 🧠 3. Analogie du restaurant (version parfaite)

## Situation normale

1. Tu commandes :  
    → salade 10€
2. Le serveur vérifie :  
    → tu as 10€ ✅
3. Il envoie la commande
4. Tu reçois une salade

👉 tout est cohérent

---

## 💣 Version vulnérable

1. Tu commandes :  
    → salade 10€
2. Le serveur vérifie ton argent :  
    → OK ✅
3. Il part vers la cuisine

⏳ **FENÊTRE CRITIQUE**

4. Tu modifies le ticket :  
    → steak 1400€
5. La cuisine prépare :  
    → steak
6. Le serveur sert :  
    → steak

---

# ⚠️ Où est la faille EXACTE ?

> ❌ ce n’est pas “le serveur a vérifié l’argent”

> ✅ c’est :  
> **il a vérifié un ticket… puis exécuté un autre**

---

# 🔬 4. Traduction backend

1. total = read(cart)  
2. if total <= credit:  
3.     process_payment()  
4. confirm_order(cart)

💥 Problème :

- étape 1 → ancien panier
- étape 4 → panier modifié

---

# 🔥 5. Nature de la vulnérabilité

> **désynchronisation entre le moment du check et le moment de l’action**

---

# 🔐 6. Atomicité

## ❌ Non atomique (vulnérable)

check → traitement → write

➡️ état intermédiaire exploitable

---

## ✅ Atomique (safe)

check + write = une seule opération

➡️ aucun moment exploitable

---

# 🔑 7. Clé d’écriture

> C’est ce qui identifie l’état modifié côté backend

Exemples :

- sessionId
- userId
- cartId
- token

👉 Une race existe seulement si :

> deux requêtes touchent la **même clé**

---

# 🏎️ 8. Rôle du timing

Le timing ne crée pas la faille.

Il permet juste :

> **d’atteindre le bon sous-état au bon moment**

---

## Timeline

T1 → checkout lit panier  
T2 → cart modifie panier  
T3 → checkout confirme

👉 Tu dois frapper à T2

---

# 🧠 9. Méthode mentale correcte

Ne pense pas :

> ❌ “comment envoyer plus vite ?”

Pense :

> ✅ “quel état backend existe brièvement avant d’être sécurisé ?”

---

# 🧪 10. Predict → Probe → Prove

## 🔍 PREDICT

Chercher :

- workflow multi-étapes
- état écrit trop tôt
- action critique (money, auth…)

---

## 💣 PROBE

Comparer :

- séquentiel
- parallèle

Chercher :

- différence de réponse
- comportement instable
- succès inattendu

---

## 🧪 PROVE

- réduire à 2 requêtes
- reproduire
- identifier la primitive
- démontrer impact

---

# 🧬 11. Signal → Primitive → Impact

|Niveau|Description|
|---|---|
|Signal|anomalie|
|Primitive|ce que le bug permet|
|Impact|exploitation réelle|

---

# 🔧 12. Comment PATCH (très important)

---

## ✅ Solution 1 — Figer l’état

👉 le panier devient immuable pendant checkout

**Analogie :**  
le ticket est scellé

---

## ✅ Solution 2 — Snapshot

👉 travailler sur une copie du panier

**Analogie :**  
photocopie du ticket

---

## ✅ Solution 3 — Transaction atomique

👉 tout en une seule opération

**Analogie :**  
le serveur garde le ticket en main jusqu’à la fin

---

## ✅ Solution 4 — Revalidation finale

👉 vérifier juste avant de confirmer

**Analogie :**  
le serveur relit le ticket avant de servir

---

# 🧠 13. Résumé mental final

- Race ≠ vitesse
- Race = **état temporaire exploitable**
- Timing = outil pour atteindre cet état
- Faille = **check ≠ use**
- Patch = **figer / atomiser / revalider**

---

# ⚡ Phrase à retenir

> **Le backend valide une chose…  
> mais agit sur une autre.**

---

Si tu veux, prochaine étape logique :  
👉 je te fais une **fiche Obsidian “signaux concrets dans Burp”** pour reconnaître ces bugs en réel (très utile pour bug bounty).