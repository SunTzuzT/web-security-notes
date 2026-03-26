
Un **endpoint** = une **porte d’entrée du backend**.

C’est une URL + une méthode HTTP que le serveur expose pour faire une action.

Exemple :

GET /api/user/842

C’est un endpoint.

POST /api/order/apply-coupon

C’est un endpoint.

---

## 🧠 Comment le voir mentalement

Imagine le backend comme un bâtiment.

Chaque endpoint = une porte.

- `/login` → porte d’authentification
    
- `/api/orders/555` → porte vers une commande
    
- `/api/admin/users` → porte admin
    

Quand tu envoies une requête, tu frappes à une porte.

---

# 📦 Structure d’un endpoint

Un endpoint est composé de :

- **Méthode HTTP** (GET, POST, PUT, DELETE)
    
- **Route** (`/api/order/555`)
    
- **Paramètres** (dans URL ou body)
    

Exemple complet :

POST /api/user/update

Avec body :

{  
  "userId": 842,  
  "email": "test@mail.com"  
}

Ça veut dire :

👉 “Backend, mets à jour cet utilisateur.”

---

# 🔍 Comment trouver les endpoints ?

1️⃣ Dans l’onglet Network du navigateur  
2️⃣ Dans Burp Proxy  
3️⃣ Dans les fichiers JS

Dans le JS, cherche :

- `fetch(`
    
- `axios(`
    
- `/api/`
    
- `"/v1/"`
    
- `"POST"`
    

---

# 🧠 Maintenant : Comment analyser un JS en 5 minutes sans te noyer

Tu ne lis pas tout.

Tu fais ça :

---

## 🔎 Étape 1 — Recherche rapide

Dans le fichier JS :

Cherche :

/api

ou

fetch(

ou

axios

Tu veux extraire les endpoints.

---

## 🔎 Étape 2 — Identifier les objets

Tu vois :

fetch("/api/order/" + orderId)

👉 Objet = Order  
👉 ID = orderId  
👉 Action = GET

---

## 🔎 Étape 3 — Identifier les champs sensibles

Tu vois :

{  
  userId: user.id,  
  role: user.role,  
  total: cart.total  
}

👉 Tu repères :

- role
    
- total
    
- userId
    

Ce sont des champs critiques.

---

## 🔎 Étape 4 — Revenir dans Burp

Tu ne restes pas dans le JS.

Tu prends les endpoints intéressants  
Et tu testes côté backend.

---

# 🎯 Exemple concret complet

JS :

axios.post("/api/subscription/upgrade", {  
  planId: selectedPlan,  
  price: plan.price  
})

Tu penses immédiatement :

- Le serveur recalcule-t-il le prix ?
    
- Puis-je changer planId ?
    
- Puis-je changer price ?
    
- Puis-je appeler ça sans passer par l’UI ?
    

---

# 🔥 Rappel important

JS = carte  
Burp = test  
Backend mindset = analyse

---

# 🧠 Résumé simple

Un endpoint est une porte du backend.  
Le JS te montre quelles portes existent.  
Burp te permet de tester si ces portes sont bien protégées.

---

# 🌐 C’est quoi `axios` ?

`axios` est une **bibliothèque JavaScript**.

Son seul rôle ici :

👉 Envoyer une requête HTTP au serveur.

C’est comme `fetch()`, mais plus pratique.

Donc :

axios.post(...)

veut juste dire :

> “Envoie une requête POST au backend.”

Rien de plus compliqué.

---

# 📌 Donc cette ligne :

axios.post("/api/subscription/upgrade", {...})

Signifie :

> Le navigateur envoie une requête POST vers le backend à l’URL `/api/subscription/upgrade`.

---

# 🧠 C’est quoi `planId` ?

`planId` = identifiant du plan d’abonnement.

Exemple :

- Plan 1 = Basic
    
- Plan 2 = Pro
    
- Plan 3 = Enterprise
    

Donc si tu choisis “Pro” dans l’UI :

planId = 2

C’est juste l’ID du plan choisi.

---

# 🧠 C’est quoi `plan.price` ?

C’est le prix du plan sélectionné, calculé côté client.

Exemple :

plan.price = 29.99

Donc le client envoie :

{  
  "planId": 2,  
  "price": 29.99  
}

---

# 🔥 Maintenant la partie sécurité

Tu ne dois pas comprendre la logique UI.

Tu dois te poser une seule question :

> Pourquoi le client envoie le prix ?

Un backend sécurisé devrait :

- Ignorer `price`
    
- Recalculer le prix à partir de `planId`
    
- Vérifier que le plan existe
    
- Vérifier que l’utilisateur peut upgrade
    

---

# 🧠 Ce qui te bloquait

Tu essayais de lire ça comme :

> “Je dois comprendre la programmation.”

Alors qu’en réalité tu dois lire ça comme :

> “Qu’est-ce que le client contrôle ?”

---

# 🎯 Ce que tu dois retenir

- `axios` = outil pour envoyer une requête
    
- `/api/subscription/upgrade` = endpoint backend
    
- `planId` = ID d’un objet (Plan)
    
- `price` = valeur sensible envoyée par le client
    

Ce qui compte, c’est :

> Le backend fait-il confiance à ces champs ?

---

Tu n’as pas besoin de maîtriser JavaScript.  
Tu dois juste savoir repérer :

- endpoint
    
- paramètres
    
- données sensibles
    

C’est tout.