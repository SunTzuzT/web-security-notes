
> **JSON = format de données utilisé pour échanger des infos entre client et serveur**

👉 Nom complet :  
**JavaScript Object Notation**

---

# ⚡ Traduction simple

> **JSON = façon standard d’écrire des données structurées**

---

# 🔥 2) Exemple concret

{  
  "username": "wiener",  
  "email": "test@test.com",  
  "isAdmin": false  
}

👉 C’est juste :

- une liste de **clés (keys)**
    
- avec des **valeurs (values)**
    

---

# 🧠 3) Structure JSON

## 📌 Toujours comme ça :

{  
  "clé": "valeur"  
}

---

## Types de valeurs :

{  
  "string": "texte",  
  "number": 123,  
  "boolean": true,  
  "null": null  
}

---

## Objets imbriqués :

{  
  "user": {  
    "id": 1,  
    "name": "wiener"  
  }  
}

---

## Listes (arrays) :

{  
  "users": ["wiener", "carlos"]  
}

---

# 🔥 4) Pourquoi JSON est important en bug bounty

👉 Parce que :

> **Presque toutes les API utilisent JSON**

---

## Exemple réel

POST /api/order  
Content-Type: application/json  
  
{  
  "productId": 5,  
  "quantity": 2  
}

👉 Tu envoies JSON  
👉 Le serveur répond JSON

---

# 🧠 5) JSON = surface d’attaque

Chaque champ = point d’entrée 💥

---

## Exemple vulnérable

{  
  "userId": 123,  
  "role": "user"  
}

👉 Tu testes :

{  
  "userId": 124,  
  "role": "admin"  
}

💥 = IDOR / privilege escalation

---

# 🔥 6) Ce que tu dois faire avec JSON

Quand tu vois JSON :

👉 tu dois tester :

- modifier valeurs
    
- ajouter champs
    
- supprimer champs
    
- changer types
    

---

## Exemple

{  
  "price": 100  
}

👉 tu testes :

{  
  "price": 1  
}

---

# 🧠 7) JSON vs HTML

|HTML|JSON|
|---|---|
|affichage|données|
|visuel|backend|
|page web|API|

---

# 🔥 8) Comment reconnaître JSON dans Burp

👉 Réponse :

{  
  "id": 1  
}

👉 Header :

Content-Type: application/json

---

# 🧠 9) Résumé simple

## JSON =

- format de données
    
- utilisé par les API
    
- structure clé/valeur
    

---

# 🧠 Phrase clé

> **“Si je vois du JSON, je suis probablement face à une API.”**

---

# 🚀 Traduction pour toi (bug bounty)

👉 JSON = là où tu attaques

Parce que :

- c’est ce que tu contrôles
    
- c’est ce que le backend traite
    
- c’est là que les devs font des erreurs