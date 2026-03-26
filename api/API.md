


1-Mass Assignment (Affectation de masse) 
2- broken object level authorization
3- hidden endpoints
4- hidden parameters



1 - 💣 Mass Assignment (Affectation de masse)  
  
## 🎯 Définition  
Le **mass assignment** est une vulnérabilité où le backend accepte et copie automatiquement des champs envoyés par le client vers un objet interne, **sans filtrer strictement les champs autorisés**.  
  
En clair :  
  
> l’application te laisse modifier plus de choses que prévu, simplement parce que le backend “remplit l’objet tout seul”.  
  
---  
  
## 🧠 Idée clé  
Le frontend peut n’afficher que **2 champs modifiables** :  
  
- `name`  
- `email`  
  
Mais l’objet backend contient peut-être aussi :  
  
- `role`  
- `isAdmin`  
- `verified`  
- `accountType`  
- `userId`  
- `discount`  
  
Si le backend fait une liaison automatique entre ta requête et l’objet interne, alors tu peux parfois envoyer ces champs sensibles même s’ils ne sont ni visibles ni documentés.  
  
---  
  
## 🍔 Analogie simple  
Imagine que tu vas à l’hôtel pour modifier une réservation.  
  
La réceptionniste te dit :  
  
- “Vous pouvez changer votre nom”  
- “Vous pouvez changer votre email”  
  
Mais au lieu de recopier seulement ces deux infos, elle prend **tout le formulaire que tu lui donnes** et le met directement dans ton dossier client.  
  
Donc toi, tu ajoutes discrètement :  
  
- `suitePresidentielle=true`  
- `vip=true`  
- `reduction=100`  
- `statut=manager`  
  
Et comme elle range tout sans vérifier, ces valeurs se retrouvent dans ton dossier.  
  
👉 **C’est exactement le mass assignment.**  
  
Le problème n’est pas juste que tu as envoyé un champ en plus.  
Le problème est que **le backend l’a accepté et recopié automatiquement**.  
  
---  
  
## 🧱 Vision technique simple  
  
## Fonctionnement normal attendu  
Le développeur veut autoriser uniquement :  
  
- `name`  
- `email`  
  
Donc il devrait faire quelque chose comme :  
  
```python  
user.name = request.json["name"]  
user.email = request.json["email"]  
```  
  
👉 Ici, seuls les champs explicitement choisis sont modifiés.  
  
---  
  
## Fonctionnement vulnérable  
Le backend fait plutôt quelque chose comme :  
  
```python  
user.update(request.json)  
```  
  
ou :  
  
```python  
bind(request.json, user)  
```  
  
ou encore :  
  
```python  
Object.assign(user, req.body)  
```  
  
👉 Là, **tous les champs reçus** peuvent être copiés dans l’objet `user`.  
  
Donc si tu envoies :  
  
```json  
{  
"name": "Chaimaa",  
"email": "test@test.com",  
"isAdmin": true  
}  
```  
  
et que `isAdmin` existe dans l’objet backend, il peut être rempli automatiquement.  
  
💥 Résultat : tu modifies un champ sensible non prévu.  
  
---  
  
## 🔥 Pourquoi c’est dangereux  
Parce que cette faille peut permettre :  
  
- **élévation de privilège**  
- **contournement de logique métier**  
- **modification de statut**  
- **manipulation de prix**  
- **changement de propriétaire**  
- **validation artificielle d’un compte**  
- **bypass de restrictions**  
  
---  
  
## 🧪 Exemple simple : élévation de privilège  
  
## Requête légitime  
```http  
POST /api/user/update  
Content-Type: application/json  
```  
  
```json  
{  
"name": "Chaimaa",  
"email": "c@test.com"  
}  
```  
  
Le frontend te laisse juste modifier ton profil.  
  
---  
  
## Test offensif  
```http  
POST /api/user/update  
Content-Type: application/json  
```  
  
```json  
{  
"name": "Chaimaa",  
"email": "c@test.com",  
"role": "admin",  
"isAdmin": true  
}  
```  
  
### Si le backend :  
- accepte la requête  
- enregistre ces champs  
- ou change ton comportement / tes accès  
  
👉 alors tu as potentiellement une **mass assignment vulnerability**.  
  
---  
  
## 🧪 Exemple métier : manipulation de commande  
  
## Requête normale  
```http  
POST /api/order  
Content-Type: application/json  
```  
  
```json  
{  
"productId": 12,  
"quantity": 1  
}  
```  
  
---  
  
## Requête testée  
```json  
{  
"productId": 12,  
"quantity": 1,  
"price": 1,  
"discount": 100,  
"status": "paid"  
}  
```  
  
### Ce que tu cherches  
- est-ce que le backend accepte `price` ?  
- est-ce qu’il accepte `discount` ?  
- est-ce qu’il accepte `status` ?  
- est-ce qu’il fait confiance à des champs qui auraient dû être recalculés côté serveur ?  
  
💥 Si oui, tu influences une décision métier sensible.  
  
---  
  
## 🧪 Exemple compte utilisateur : validation artificielle  
  
## Requête normale  
```json  
{  
"username": "chaimaa",  
"phone": "0600000000"  
}  
```  
  
## Requête offensive  
```json  
{  
"username": "chaimaa",  
"phone": "0600000000",  
"verified": true  
}  
```  
  
### Impact possible  
- compte considéré comme validé  
- bypass d’étape de vérification  
- accès à des fonctions réservées  
  
---  
  
## 🧠 Comment reconnaître cette faille  
Tu la soupçonnes quand :  
  
- l’API reçoit un **objet JSON complet**  
- la requête ressemble à une mise à jour d’objet (`update`, `edit`, `profile`, `account`)  
- l’application semble utiliser un framework qui fait du binding automatique  
- le backend semble accepter des champs “en trop”  
- certains champs changent alors qu’ils ne sont pas censés être modifiables  
  
---  
  
## 🔍 Ce que tu dois penser quand tu vois une requête JSON  
Quand tu interceptes un body comme :  
  
```json  
{  
"name": "Chaimaa",  
"email": "c@test.com"  
}  
```  
  
tu ne dois pas juste lire :  
  
> “je peux modifier name et email”  
  
Tu dois penser :  
  
> “Quel est l’objet backend complet derrière cette requête ?”  
  
Peut-être qu’il contient aussi :  
  
- `role`  
- `isAdmin`  
- `verified`  
- `status`  
- `userId`  
- `accountType`  
- `balance`  
- `discount`  
- `permissions`  
  
👉 Le vrai raisonnement est là.  
  
---  
  
## 🧭 Méthodologie de test  
  
## 1. Identifier un endpoint qui met à jour un objet  
Les cibles typiques :  
  
- `/api/user/update`  
- `/api/profile/edit`  
- `/api/account`  
- `/api/order/update`  
- `/api/settings`  
- `/api/admin/user/edit`  
  
Ces endpoints sont intéressants car ils manipulent souvent un objet complet.  
  
---  
  
## 2. Observer les champs déjà envoyés  
Exemple :  
  
```json  
{  
"name": "Chaimaa",  
"email": "c@test.com"  
}  
```  
  
Tu repères :  
- la structure JSON  
- le type d’objet manipulé  
- la logique métier liée  
  
---  
  
## 3. Imaginer les champs backend probables  
Toujours à partir de l’objet visé.  
  
### Si c’est un `user`  
penser à :  
- `role`  
- `isAdmin`  
- `verified`  
- `status`  
- `permissions`  
- `accountType`  
  
### Si c’est un `order`  
penser à :  
- `price`  
- `discount`  
- `status`  
- `paid`  
- `currency`  
- `ownerId`  
  
### Si c’est un `product`  
penser à :  
- `isPublished`  
- `hidden`  
- `price`  
- `stock`  
- `sellerId`  
  
---  
  ## 🧪 Test progressif d’un champ sensible (`isAdmin`)

### 1. Vérifier que le champ est accepté
```http
PATCH /api/users/123
Content-Type: application/json
```

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": false
}
```

👉 Objectif : voir si le backend accepte déjà la présence du champ sans le rejeter.

---

### 2. Vérifier que le champ est réellement traité
```http
PATCH /api/users/123
Content-Type: application/json
```

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": "foo"
}
```

👉 Objectif : si la réponse change (erreur, taille, contenu, comportement), cela suggère que `isAdmin` entre réellement dans la logique backend et n’est pas simplement ignoré.

---

### 3. Tenter l’exploitation
```http
PATCH /api/users/123
Content-Type: application/json
```

```json
{
  "username": "wiener",
  "email": "wiener@example.com",
  "isAdmin": true
}
```

👉 Objectif : tenter de modifier un champ sensible ; il faut ensuite vérifier l’effet réel dans l’application, par exemple l’accès à des fonctionnalités d’administration.

---

## 🧠 Logique de la méthode
- `false` → test de présence du champ
- `"foo"` → test de traitement / validation
- `true` → test d’exploitation

> Le but n’est pas seulement de voir un `200 OK`, mais de confirmer que le champ sensible est bien pris en compte par le backend et qu’il produit un effet réel.
  
  
  ## 🔍 Identifier les champs cachés via les objets renvoyés par l’API  

Pour repérer des champs potentiellement exploitables en **mass assignment**, compare les champs envoyés dans une requête de mise à jour avec ceux renvoyés par un endpoint de lecture : tout champ présent dans l’objet complet mais absent de la requête initiale devient un candidat plausible à tester.  
  
Exemple :  
  
```http  
PATCH /api/users/123  
Content-Type: application/json  
```  
  
```json  
{  
"username": "wiener",  
"email": "wiener@example.com"  
}  
```  
  
```http  
GET /api/users/123  
```  
  
```json  
{  
"id": 123,  
"username": "wiener",  
"email": "wiener@example.com",  
"isAdmin": false  
}  
```  
  
Ici, `id` et `isAdmin` apparaissent dans l’objet complet mais pas dans la requête de mise à jour initiale : ce sont donc des champs internes plausibles à ajouter dans le `PATCH` pour tester un éventuel **mass assignment**.
## 4. Ajouter un champ sensible  
Tu modifies la requête en ajoutant un seul champ à la fois.  
  
Exemple :  
```json  
{  
"name": "Chaimaa",  
"email": "c@test.com",  
"isAdmin": true  
}  
```  
  
👉 Un champ à la fois = plus facile à analyser.  
  
---  
  
## 5. Observer la réponse  
Tu regardes :  
  
- code HTTP  
- taille  
- message retourné  
- contenu JSON  
- différence de comportement dans l’application  
  
---  
  
## 6. Vérifier l’effet réel  
Le plus important :  
  
> est-ce que la valeur a vraiment été prise en compte ?  
  
Par exemple :  
- ton rôle a-t-il changé ?  
- un accès admin est-il apparu ?  
- la commande est-elle “paid” ?  
- le produit est-il maintenant caché / publié ?  
- l’utilisateur est-il “verified” ?  
  
👉 Une mass assignment n’est pas juste “un champ accepté”.  
👉 C’est **un champ sensible accepté avec effet réel**.  
  
---  
  
## ⚠️ Ce qu’il ne faut surtout pas rater  
  
## 1. Le backend peut ignorer silencieusement  
Parfois tu envoies :  
  
```json  
"isAdmin": true  
```  
  
et la réponse reste `200 OK`, mais le champ n’a pas été appliqué.  
  
👉 Donc :  
- ne jamais conclure trop vite  
- toujours vérifier l’effet réel ensuite  
  
---  
  
## 2. L’impact peut être indirect  
Parfois tu ne deviens pas admin immédiatement, mais :  
- un champ de validation change  
- une étape de workflow saute  
- un statut passe à “approved”  
- un flag interne s’active  
  
👉 Il faut penser au **métier**, pas seulement à `isAdmin`.  
  
---  
  
## 3. Les noms peuvent varier  
Un développeur n’utilise pas forcément :  
- `isAdmin`  
  
Il peut utiliser :  
- `role`  
- `admin`  
- `accessLevel`  
- `permission`  
- `group`  
- `accountType`  
  
👉 Toujours penser aux synonymes.  
  
---  
  
## 4. Certains champs sensibles sont numériques  
Exemple :  
- `roleId: 1`  
- `accessLevel: 10`  
- `discount: 100`  
- `price: 1`  
  
Ne pense pas uniquement en booléens (`true/false`).  
  
---  
  
## 5. La faille peut exister même sans réponse explicite  
Parfois la réponse ne montre rien, mais la valeur est bien enregistrée.  
  
👉 Donc il faut parfois :  
- relire la ressource ensuite  
- recharger le profil  
- observer l’UI  
- appeler un autre endpoint pour confirmer  
  
---  
  
## 🧩 Différence avec un simple paramètre caché  
Un **paramètre caché** = champ non documenté accepté par l’application.  
  
Le **mass assignment** = cas particulier où ce champ est accepté parce que le backend fait une **liaison automatique** vers un objet interne.  
  
Donc :  
  
> le mass assignment crée souvent des paramètres cachés exploitables.  
  
---  
  
## 🧠 Lien avec ton modèle mental  
Toujours raisonner comme ça :  
  
```text  
Requête envoyée -> Binding automatique -> Objet backend modifié -> Décision backend impactée  
```  
  
Le vrai point critique est ici :  
  
> **Quel champ interne puis-je influencer sans que le développeur l’ait prévu ?**  
  
---  
  
## 💥 Impacts typiques  
  
### Compte utilisateur  
- devenir admin  
- devenir “verified”  
- changer de rôle  
- changer d’organisation / tenant  
  
### E-commerce  
- modifier le prix  
- appliquer une réduction  
- marquer une commande comme payée  
- changer le propriétaire d’une commande  
  
### Plateformes internes / SaaS  
- changer les permissions  
- s’ajouter à une équipe  
- activer des fonctions premium  
- changer le statut d’un ticket / document / workflow  
  
---  
  
## 🧠 Résumé final  
- Le mass assignment arrive quand le backend copie automatiquement les champs de la requête vers un objet interne  
- Tu peux alors envoyer des champs non prévus mais existants dans ce modèle  
- Si ces champs sont sensibles et réellement pris en compte, tu as une vulnérabilité  
- Le vrai enjeu n’est pas d’ajouter “n’importe quel champ”  
- Le vrai enjeu est d’identifier **les champs internes qui pilotent une décision sensible**  
  
---  
  
## 🧠 Formule mentale à retenir  
> **Le frontend montre 2 champs.  
Le backend en possède peut-être 10.  
Le framework les remplit tous si personne ne filtre.**  
  
---  
  
## ✅ Mini checklist terrain  
  
### Quand je vois un body JSON de mise à jour :  
- quel objet est modifié ?  
- quels champs backend probables existent ?  
- quels champs sensibles pilotent la logique ?  
- puis-je en ajouter un ?  
- la réponse change-t-elle ?  
- l’effet réel est-il visible ?  
  
---  
  
## 🚀 Phrase bug bounty à garder en tête  
> **Je ne teste pas juste des champs en plus.  
Je cherche quels attributs internes du modèle backend influencent la sécurité ou la logique métier.**




2- broken object level authorization
3- hidden endpoints
4- hidden parameters
