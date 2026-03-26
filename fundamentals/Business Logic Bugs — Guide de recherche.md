
## Définition

Un **business logic bug** apparaît lorsque l’application :

accepte une action logique incorrecte

Ce n’est pas une faille technique classique (SQLi, XSS).

C’est un problème dans **la logique du système**.

---

# 🏬 Analogie (entrepôt)

Imagine un magasin.

Normalement :

1 produit → 1 paiement → livraison

Mais si la logique est mauvaise :

payer 0€  
et recevoir le produit

Le système fonctionne techniquement…  
mais **la logique métier est cassée**.

---

# 📌 Différence avec les failles techniques

|type|exemple|
|---|---|
|SQLi|injection dans requête|
|XSS|exécution JS|
|IDOR|accès à un autre utilisateur|
|Business logic|système manipulable|

---

# 🧠 Mentalité pour trouver ces bugs

Ne pas penser :

payload

mais penser :

workflow

---

# 🔎 Étudier le workflow

Observer les étapes :

login  
↓  
panier  
↓  
paiement  
↓  
commande

Puis tester :

ordre différent

---

# ⚠️ Types de Business Logic Bugs

## 1️⃣ Manipulation de prix

Changer :

{  
 "price": 100  
}

en

{  
 "price": 1  
}

---

## Analogie

Le client change l’étiquette du produit.

La caisse accepte le prix modifié.

---

# 2️⃣ Bypass d'étapes

Exemple workflow :

panier  
↓  
paiement  
↓  
confirmation

Essayer :

aller directement à confirmation

---

## Analogie

Passer la caisse **sans payer**.

---

# 3️⃣ Quantité négative

Exemple :

{  
 "quantity": -1  
}

Peut provoquer :

remboursement

---

# 4️⃣ Coupons multiples

Normalement :

1 coupon par commande

Mais tester :

coupon1 + coupon2 + coupon3

---

# 5️⃣ Race conditions

Envoyer plusieurs requêtes simultanément :

redeem coupon  
redeem coupon  
redeem coupon

Résultat possible :

coupon utilisé plusieurs fois

---

# 6️⃣ Limites mal contrôlées

Exemple :

max withdraw = 100€

Tester :

99  
100  
101  
1000

---

# 🔑 Paramètres sensibles

Toujours surveiller :

price  
discount  
quantity  
role  
userId  
balance

---

# 🧪 Méthode de recherche

## 1️⃣ Cartographier l'application

Identifier :

endpoints  
API  
paramètres

---

## 2️⃣ Comprendre la logique

Se demander :

qu'est-ce que l'application essaye d'empêcher ?

---

## 3️⃣ Tester les limites

Tester :

0  
-1  
999999  
null

---

## 4️⃣ Changer l'ordre

Tester :

step 3 avant step 2

---

# 🧰 Outils utiles

|outil|usage|
|---|---|
|Burp Repeater|modifier requêtes|
|Burp Intruder|tester paramètres|
|Turbo Intruder|race conditions|

---

# 🧠 Questions que se posent les hunters

Toujours se demander :

si j'étais malveillant  
comment abuser du système ?

---

# 📌 Signaux d’un business logic bug

- workflow complexe
    
- nombreuses étapes
    
- calculs financiers
    
- promotions
    
- coupons
    
- transactions
    

---

# 💰 Pourquoi ces bugs sont précieux

Parce qu’ils impactent directement :

argent  
transactions  
inventaire

Les programmes bug bounty les classent souvent :

high  
critical

---

# 🎯 Objectif du pentester

Comprendre :

comment le système devrait fonctionner

puis chercher :

comment le détourner

---

# Conclusion

Un business logic bug consiste à :

faire fonctionner le système  
contre lui-même

Sans casser la technique, mais en cassant :

la logique métier
