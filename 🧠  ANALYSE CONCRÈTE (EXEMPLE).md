
## 🎯 Contexte

Application SaaS e-commerce  
Fonction : Appliquer un coupon de réduction

---

## 1️⃣ Identification

**Rubrique :** Checkout  
**Endpoint :** `POST /api/order/apply-coupon`  
**Méthode :** POST

---

## 2️⃣ Structure fonctionnelle

**Objet :** Order  
**Action :** Apply coupon  
**ID utilisé :** `orderId`

Requête interceptée :

{  
  "orderId": 8821,  
  "coupon": "PROMO50",  
  "total": 49.99  
}

---

## 3️⃣ Promesse du serveur (Décision critique)

Le serveur doit garantir :

- L’utilisateur est authentifié
    
- La commande 8821 appartient à l’utilisateur
    
- Le coupon est valide
    
- Le total est recalculé côté serveur
    
- Le coupon n’est pas utilisé deux fois
    

👉 Promesse formulée :

> “Le serveur doit recalculer le total et vérifier que la commande appartient à l’utilisateur connecté.”

---

## 4️⃣ Visualisation backend (pipeline mental)

Auth  
→ CheckOwnership(orderId)  
→ ValidateCoupon  
→ RecalculateTotal  
→ UpdateOrder  
→ Save  
→ Respond

---

## 5️⃣ Tests appliqués

### 🔹 Test 1 — Ownership

Modifier `orderId` → 8822

Résultat : 403  
→ Ownership vérifié ✔️

---

### 🔹 Test 2 — Manipulation total

Changer :

"total": 1.00

Résultat :  
Réponse 200  
Total retourné = 12.50

→ Le serveur recalcul ✔️

---

### 🔹 Test 3 — Double application coupon

Rejouer la requête 2 fois

Résultat :  
Total diminue encore

💥 Hypothèse validée : coupon appliqué plusieurs fois

---

## 6️⃣ Analyse

Pipeline probablement :

- Auth OK
    
- Ownership OK
    
- Coupon validé
    
- MAIS absence de vérification “already applied”
    

Bug potentiel :  
Business Logic flaw (double discount)

---

## 7️⃣ Conclusion technique

Type : Business Logic  
Impact : Réduction illimitée  
Gravité : Élevée (perte financière)

---

# 🔥 Ce que tu dois observer dans cet exemple

On n’a pas cherché un payload.

On a :

- Identifié l’objet
    
- Identifié l’action
    
- Identifié l’ID
    
- Formulé la promesse
    
- Testé chaque couche
    

C’est méthodique.