
## 🎯 Définition  
Un **endpoint caché** = une route API qui existe côté backend mais qui :  
- n’est pas visible directement dans l’interface  
- n’est pas documentée  
- n’apparaît pas forcément dans les appels normaux du frontend  
  
---  
  
## ⚡ Idée clé  
> Si tu trouves un endpoint, tu peux souvent deviner les autres.  
  
Les APIs suivent souvent une logique de nommage.  
  
### Exemple  
Si tu trouves :  
```http  
PUT /api/user/update  
```  
  
tu peux soupçonner l’existence de :  
```http  
DELETE /api/user/delete  
POST /api/user/create  
GET /api/user/details  
GET /api/user/list  
PATCH /api/user/edit  
```  
  
---  
  
## 🍔 Analogie  
Imagine un immeuble.  
  
Tu vois une porte :  
- **Appartement 201**  
  
Tu peux logiquement te dire qu’il existe peut-être aussi :  
- 202  
- 203  
- 204  
  
Tu ne les vois pas encore, mais la structure te donne une piste.  
  
👉 En API, c’est pareil :  
si tu vois une route avec une certaine logique, tu cherches les autres portes du même couloir.  
  
---  
  
## 🔥 Pourquoi c’est important en bug bounty  
Les endpoints cachés sont souvent :  
- oubliés  
- mal protégés  
- non testés  
- réservés à l’admin ou à des usages internes  
  
Donc ils peuvent mener à :  
- IDOR / BOLA  
- broken access control  
- actions admin accessibles  
- suppression / modification non autorisée  
- exposition de données  
  
---  
  
## 🧭 Méthodologie complète  
  
## 1. Trouver un premier endpoint visible  
Point de départ :  
- HTTP history  
- JS  
- trafic frontend  
- docs API  
- erreurs  
- requêtes XHR / fetch / axios  
  
### Exemple  
```http  
PUT /api/user/update  
```  
  
---  
  
## 2. Découper mentalement l’endpoint  
Toujours le lire comme ça :  
  
```text  
/api / user / update  
```  
  
### Questions à se poser  
- **Objet** : ici `user`  
- **Action** : ici `update`  
- **Convention de nommage** : quel pattern est utilisé ?  
  
---  
  
## 3. Dériver les endpoints logiques  
À partir de `update`, penser :  
- create  
- add  
- delete  
- remove  
- edit  
- details  
- info  
- list  
- search  
- export  
  
À partir de `user`, penser aussi aux autres objets proches :  
- users  
- profile  
- account  
- admin  
- settings  
  
---  
  
## 4. Fuzzer intelligemment  
Le but n’est pas de spammer 10 000 mots au hasard.  
  
Le but est de tester :  
- des mots génériques API  
- des mots liés au métier de l’application  
  
### Exemple e-commerce  
- cart  
- checkout  
- order  
- coupon  
- refund  
- payment  
- invoice  
  
### Exemple user/account  
- profile  
- password  
- role  
- permissions  
- settings  
- delete  
  
---  
  
## 5. Observer les réponses  
Ne jamais regarder seulement si “ça marche ou pas”.  
  
Toujours comparer :  
  
- **code HTTP**  
- **taille**  
- **contenu**  
- **temps de réponse**  
  
### Exemple  
```http  
/api/user/update -> 200  
/api/user/delete -> 403  
/api/user/remove -> 404  
```  
  
### Interprétation  
- `update` : accessible  
- `delete` : existe mais protégé  
- `remove` : probablement inexistant  
  
👉 Le plus intéressant ici = `delete`  
  
---  
  
## 6. Poser l’hypothèse backend  
Chaque endpoint trouvé doit déclencher cette question :  
  
> Quelle action sensible le backend effectue ici ?  
  
### Exemple  
```http  
/api/user/delete  
```  
  
Questions :  
- qui a le droit d’appeler ça ?  
- est-ce que le rôle est vérifié ?  
- est-ce qu’un simple utilisateur peut l’atteindre ?  
- est-ce que l’ID ciblé est bien revalidé côté serveur ?  
  
---  
  
## 7. Tester ensuite la sécurité de l’endpoint  
Une fois l’endpoint trouvé, tu ne t’arrêtes pas là.  
  
Tu testes :  
- authentification  
- autorisation  
- changement d’ID  
- changement de méthode HTTP  
- ajout de paramètres cachés  
- comportement avec un autre compte  
  
---  
  
## 🛠️ Comment les trouver concrètement  
  
## A. Via HTTP History  
Regarder :  
- toutes les requêtes déclenchées par l’UI  
- les variations d’URL  
- les endpoints appelés après une action précise  
  
---  
  
## B. Via les fichiers JavaScript  
Chercher :  
- routes hardcodées  
- noms d’objets  
- fonctions API  
- chemins internes  
- endpoints commentés / oubliés  
  
---  
  
## C. Via Burp Intruder  
Tu prends une partie du chemin et tu la remplaces avec une wordlist.  
  
### Exemple  
```http  
/api/user/§update§  
```  
  
Payloads :  
- delete  
- create  
- details  
- list  
- info  
- password  
- role  
  
---  
  
## D. Via Content Discovery  
Même logique qu’un brute force de répertoires, mais appliquée aux routes API.  
  
---  
  
## E. Via la logique métier  
Toujours se demander :  
  
> Si l’application permet une action, quelle route backend doit exister pour la faire ?  
  
### Exemple  
Si un utilisateur peut :  
- modifier son profil  
- changer son mot de passe  
- supprimer son compte  
- voir ses commandes  
  
alors des endpoints liés doivent probablement exister.  
  
---  
  
## 🧠 Comment ne rien oublier  
  
## La règle simple  
Toujours raisonner en :  
  
```text  
Objet -> Actions possibles -> Contrôles attendus  
```  
  
### Exemple  
Objet :  
- user  
  
Actions possibles :  
- create  
- read  
- update  
- delete  
- list  
- role  
- password  
- settings  
  
Contrôles attendus :  
- auth  
- ownership  
- rôle admin si sensible  
- validation serveur  
  
---  
  
## Checklist mentale  
  
### 1. Quel est l’objet ?  
- user  
- order  
- invoice  
- product  
- cart  
- admin  
  
### 2. Quelles actions normales existent ?  
- create  
- read  
- update  
- delete  
- list  
- export  
- search  
  
### 3. Quelles actions sensibles existent peut-être ?  
- delete  
- role change  
- refund  
- approve  
- disable  
- invite  
- reset password  
  
### 4. Qui devrait pouvoir faire ça ?  
- owner ?  
- admin ?  
- support ?  
- système interne seulement ?  
  
### 5. Quelle erreur de sécurité peut exister ?  
- endpoint exposé  
- contrôle d’accès absent  
- méthode alternative oubliée  
- endpoint interne réutilisable  
  
---  
  
## 🧩 Ce qu’il ne faut surtout pas rater  
  
### 1. Les synonymes  
Un dev peut utiliser :  
- delete  
- remove  
- destroy  
  
ou :  
- details  
- info  
- profile  
  
👉 Toujours penser aux variantes.  
  
---  
  
## 2. Le pluriel / singulier  
- `/user/`  
- `/users/`  
  
---  
  
## 3. Les versions API  
- `/api/v1/user/update`  
- `/api/v2/user/update`  
  
---  
  
## 4. Les méthodes HTTP différentes  
Même chemin, comportement différent :  
  
```http  
GET /api/user  
POST /api/user  
DELETE /api/user  
```  
  
👉 L’endpoint n’est pas seulement le chemin.  
👉 Le couple important = **méthode + chemin**  
  
---  
  
## 5. Les endpoints trouvés mais “bloqués”  
Un `403` est souvent plus intéressant qu’un `404`.  
  
Pourquoi ?  
Parce qu’il te dit :  
> “la porte existe”  
  
---  
  
## 🧪 Exemple de raisonnement complet  
  
### Endpoint visible  
```http  
PUT /api/user/update  
```  
  
### Dérivations logiques  
```http  
DELETE /api/user/delete  
GET /api/user/details  
POST /api/user/create  
GET /api/user/list  
PATCH /api/user/password  
```  
  
### Lecture des réponses  
- `update` -> 200  
- `delete` -> 403  
- `details` -> 200  
- `list` -> 401  
- `remove` -> 404  
  
### Ce que tu comprends  
- `delete` existe, accès refusé  
- `list` existe, auth requise  
- `details` existe et mérite d’être creusé  
- `remove` semble faux  
  
### Étape suivante  
Tester :  
- autre user ID  
- autre méthode  
- paramètres cachés  
- autre rôle / compte  
  
---  
  
## 🧠 Vision offensive  
Le vrai objectif n’est pas juste de “trouver une route”.  
  
Le vrai objectif est de trouver :  
  
> une **action backend sensible** accessible d’une manière non prévue  
  
Donc un endpoint caché n’a de valeur que s’il mène à :  
- une donnée intéressante  
- une action intéressante  
- un contrôle d’accès faible  
- une logique métier cassable  
  
---  
  
## 🚀 Résumé final  
- Un endpoint caché = une route backend non visible ou non documentée  
- Tu les trouves en partant d’un endpoint connu  
- Tu dérives selon la logique de nommage  
- Tu observes les différences de réponse  
- Tu raisonnes toujours en :  
- objet  
- action  
- contrôle attendu  
- Le but final = trouver une décision backend sensible mal protégée  
  
---  
  
## 🧠 Formule mentale à retenir  
> **Je ne cherche pas juste des routes.  
Je cherche des actions backend oubliées, exposées ou mal protégées.**
