
# 🔥 1. Vision globale

```
Utilisateur → HTTP → Serveur (vérifie + logique) → SQL → Base de données
                                         ↓
Utilisateur ← JSON ← Serveur ← données
```

---

# ⚡ 2. Les rôles de chaque élément

---

## 👤 Utilisateur (navigateur)

- Clique, interagit avec le site
    
- Envoie des requêtes HTTP
    

Exemple :

```
GET /profile
POST /order
```

---

## 🌐 HTTP (communication)

> HTTP = le transport des requêtes

- Permet au client de parler au serveur
    
- Contient :
    
    - URL
        
    - Headers
        
    - Body (parfois JSON)
        

---

## 🧾 JSON (format de données)

> JSON = format utilisé pour envoyer des données

Exemple :

```
{
  "productId": 5,
  "quantity": 2
}
```

⚠️ Important :

- JSON ≠ HTTP
    
- JSON ≠ API
    

---

## 🧠 Serveur (Backend : Python, Java, etc.)

> Le serveur est le cerveau de l’application

Il :

- reçoit la requête HTTP
    
- comprend la demande
    
- applique la logique métier
    
- **vérifie la sécurité 🔥**
    
- décide quoi faire
    
- interagit avec la base de données
    

---

## 🗄️ Base de données (BDD)

> Stocke les données

- ne connaît pas HTTP
    
- ne connaît pas JSON
    
- parle uniquement en SQL
    

---

## 🧾 SQL

> Langage utilisé pour interroger la base de données

Exemple :

```
SELECT * FROM users WHERE id = 1;
```

---

# 🔁 3. Flow complet (exemple réel)

---

## Étape 1 — Requête utilisateur

```
POST /api/order
Content-Type: application/json
```

```
{
  "productId": 5,
  "quantity": 2
}
```

---

## Étape 2 — Réception serveur

👉 Le serveur reçoit la requête  
👉 Il ne doit **JAMAIS faire confiance**

---

## 🔐 Étape 3 — Vérifications critiques (TRÈS IMPORTANT)

Le serveur doit vérifier :

### ✔️ Authentification (Qui es-tu ?)

- session
    
- token
    

---

### ✔️ Autorisation (As-tu le droit ?)

- accès à cette ressource ?
    
- action autorisée ?
    

---

### ✔️ Validation des données

- type correct ?
    
- valeur logique ?
    
- champs autorisés ?
    

---

## Étape 4 — Interaction avec la BDD

```
SELECT price FROM products WHERE id = 5;
```

---

## Étape 5 — Réponse de la BDD

```
price = 100
```

---

## Étape 6 — Réponse serveur

```
{
  "total": 200
}
```

---

# 🔥 4. Où apparaissent les failles

```
Client envoie → Serveur ne vérifie pas ❌ → BDD → 💥 FAILLE
```

---

# ⚠️ 5. Règle fondamentale (ULTRA IMPORTANTE)

> **Le client n’est jamais fiable**

Pourquoi ?

- requêtes modifiables (Burp)
    
- JSON falsifiable
    
- UI contournable
    

---

# 🧠 6. Ce qu’il faut absolument retenir

---

## ❌ JSON ≠ HTTP

- HTTP = transport
    
- JSON = données
    

---

## ❌ API ≠ JSON

> API = endpoints + logique backend

---

## ❌ La BDD ne parle pas HTTP

> Elle parle uniquement SQL

---

## ❌ Le serveur ne parle pas à lui-même

> Il exécute du code et orchestre les échanges

---

# 🔥 7. Vision mentale simple

```
Client → demande
Serveur → vérifie
BDD → répond
Serveur → renvoie
```

---

# 🧠 8. Réflexe bug bounty (TRÈS IMPORTANT)

Quand tu vois une requête :

👉 tu dois te demander :

- Puis-je changer un ID ?
    
- Puis-je modifier une valeur ?
    
- Puis-je ajouter un champ ?
    
- Le serveur vérifie-t-il vraiment ?
    

---

# 🎯 9. Ce que tu attaques réellement

- les requêtes HTTP
    
- le JSON envoyé
    
- la logique du serveur
    

---

# 💥 10. Exemple de faille

```
{
  "price": 1
}
```

👉 Si accepté :

💥 manipulation de logique backend

---

# 🧠 11. Phrase clé (à retenir absolument)

> **Le client demande, le serveur vérifie, la base de données répond.**

---

# 🚀 12. Conclusion

Comprendre ce flow permet de :

- savoir où attaquer
    
- comprendre les vulnérabilités
    
- raisonner comme un hacker
    

---

# ⚡ Insight final

> **Chaque faille est un moment où le serveur a fait confiance au client**

---