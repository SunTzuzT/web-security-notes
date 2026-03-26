
## 🎯 Objectif du lab  
Identifier une vulnérabilité de **mass assignment** sur un endpoint API de checkout en manipulant un objet JSON accepté par le backend.  
  
---  
  
## 🧠 Idée clé  
Le backend expose et accepte un **objet de checkout complet**.  
Comme certains champs sensibles sont déjà présents dans cet objet, il est possible de **modifier directement leur valeur** sans devoir forcément inventer de nouveaux paramètres.  
  
> Ici, la faille ne vient pas forcément d’un champ caché ajouté, mais du fait que le backend accepte un objet contenant déjà un champ métier sensible modifiable.  
  
---  
  
## 🔍 Étapes suivies  
  
### 1. Découverte de l’endpoint  
Un endpoint `GET /api/checkout` a été identifié.  
  
La réponse renvoyait un objet JSON du type :  
  
```json  
{  
"chosen_discount": {  
"percentage": 0  
},  
"chosen_products": [  
{  
"product_id": "1",  
"name": "Lightweight \"l33t\" Leather Jacket",  
"quantity": 3,  
"item_price": 133700  
}  
]  
}  
```  
  
### Ce que ça révélait  
- la structure de l’objet backend de checkout  
- la présence d’un champ sensible : `chosen_discount.percentage`  
- la possibilité que cet objet soit repris tel quel lors du `POST`  
  
---  
  
### 2. Vérification des méthodes autorisées  
Un test avec `OPTIONS` a montré que seules les méthodes suivantes étaient acceptées :  
- `GET`  
- `POST`  
  
### Conclusion  
L’absence de `PATCH` ou `PUT` n’empêche **pas** une faille de mass assignment.  
  
> Le mass assignment dépend du **binding automatique des champs envoyés**, pas de la méthode HTTP utilisée.  
  
---  
  
### 3. Premier test sur `POST /api/checkout`  
Un body vide ou mal formé a d’abord provoqué des erreurs utiles :  
  
#### Erreur de parsing JSON  
Exemple :  
- `Unexpected ')' at line 1, column 11`  
  
### Interprétation  
Le backend parse bien le body en JSON.  
  
#### Erreur de structure  
Avec un JSON valide mais vide :  
```json  
{}  
```  
  
le backend a répondu avec un message du type :  
- `Key chosen_products: undefined is not an array`  
  
### Interprétation  
Le backend attend explicitement :  
- une clé `chosen_products`  
- de type tableau  
  
> Les messages d’erreur ont permis de déduire la structure attendue par le backend.  
  
---  
  
### 4. Envoi d’un body minimal valide  
Un body minimal cohérent a ensuite été envoyé :  
  
```json  
{  
"chosen_products": [  
{  
"product_id": "1",  
"quantity": 3  
}  
]  
}  
```  
  
La réponse a été :  
- `201 Created`  
  
### Interprétation  
Le backend accepte bien un objet JSON client pour le checkout.  
  
> À ce stade, la surface de mass assignment était confirmée.  
  
---  
  
### 5. Exploitation  
Le champ sensible déjà visible dans l’objet a été modifié :  
  
```json  
{  
"chosen_discount": {  
"percentage": 100  
},  
"chosen_products": [  
{  
"product_id": "1",  
"quantity": 3  
}  
]  
}  
```  
  
### Résultat  
Le lab a été résolu.  
  
---  
  
## ✅ Pourquoi c’est bien du mass assignment  
Parce que le backend :  
- accepte un objet JSON structuré envoyé par le client  
- contient un champ métier sensible dans cet objet  
- prend en compte la valeur modifiée sans revalidation stricte  
  
Le problème est donc :  
  
> **le backend lie et traite un objet contrôlé par le client contenant un champ sensible (`chosen_discount.percentage`)**  
  
---  
  
## ⚠️ Point important  
Le panier s’est vidé après l’exploitation.  
  
### Interprétation correcte  
Le `POST /api/checkout` ne servait pas à afficher ou mettre à jour le panier, mais à **finaliser la commande**.  
Il est donc normal que :  
- l’objet de checkout soit consommé  
- puis que le panier soit vidé  
  
> Le succès de l’exploitation se voit dans le **résultat du workflow**, pas forcément dans une modification visuelle du panier avant validation.  
  
---  
  
## 🍔 Analogie  
C’est comme si un hôtel te permettait de “modifier ta fiche client”, mais utilisait en réalité une **fiche complète** contenant aussi un champ :  
- `réduction = 0%`  
  
Comme le système reprend la fiche telle quelle sans vérifier champ par champ, tu peux simplement remplacer :  
- `0%` par `100%`  
  
et le système applique la nouvelle valeur au moment du paiement.  
  
---  
  
## 🧩 Ce que ce lab apprend vraiment  
  
### 1. Le mass assignment ne dépend pas de `PATCH`  
Un `POST` peut parfaitement être vulnérable si le backend bind automatiquement l’objet reçu.  
  
### 2. Un objet retourné par un `GET` peut révéler la structure interne utile  
Le `GET /api/checkout` a servi de carte du modèle backend.  
  
### 3. Les messages d’erreur aident à reconstruire la structure attendue  
- erreur de parsing → JSON invalide  
- erreur sur une clé → JSON valide mais structure incorrecte  
  
### 4. Il n’est pas toujours nécessaire d’ajouter un champ caché  
Si le champ sensible est déjà présent dans l’objet, il suffit parfois de **modifier sa valeur**.  
  
---  
  
## 🧠 Formule mentale à retenir  
> **Le frontend ou l’API me montre un objet complet.  
Si le backend le réutilise sans filtrer strictement les champs sensibles, je peux manipuler la logique métier directement dans cet objet.**  
  
---  
  
## ✅ Méthode retenue  
1. Identifier un endpoint lisant un objet complet  
2. Lire la structure via `GET`  
3. Tester le `POST`  
4. Corriger la syntaxe JSON  
5. Déduire la structure attendue grâce aux erreurs  
6. Envoyer un body valide minimal  
7. Modifier un champ sensible  
8. Vérifier l’effet réel dans le workflow  
  
---  
  
## 🚀 Conclusion  
Ce lab montre une forme très pédagogique de **mass assignment** :  
- l’objet de checkout est exposé  
- le backend accepte cet objet côté client  
- un champ métier sensible (`chosen_discount.percentage`) peut être modifié  
- la logique de paiement est impactée  
  
> **Le vrai réflexe à garder : quand une API renvoie un objet complet puis accepte un objet similaire en entrée, il faut immédiatement chercher quels champs sensibles de cet objet peuvent être manipulés.**