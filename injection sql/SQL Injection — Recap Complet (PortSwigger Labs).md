
# 📌 1. Qu’est-ce qu’une SQL Injection ?

Une **SQL Injection (SQLi)** est une faille où une entrée utilisateur permet de **modifier une requête SQL côté serveur**.

👉 Cause principale :

- concaténation directe des entrées utilisateur dans le SQL
    

```
SELECT * FROM users WHERE username = '<input>'
```

---

# 🎯 Objectif d’une SQLi

- Lire des données (dump DB)
    
- Bypass login
    
- Modifier logique métier
    
- Exfiltrer des données sensibles
    

---

# 🧪 2. Détection d’une SQLi

## 🔍 Signaux classiques

- erreurs SQL (500, stacktrace)
    
- comportement anormal
    
- différences de réponse
    

## 🧠 Tests de base

```
'
''
' OR '1'='1
'--
```

---

# 📂 3. Types de SQL Injection

---

## ⚡ A. In-band SQLi (visible)

### 1. Error-based

👉 le serveur affiche les erreurs SQL

---

### 2. UNION-based

👉 permet de récupérer des données d’autres tables

```
' UNION SELECT null, username, password FROM users--
```

### 🔑 Étapes :

1. Trouver le nombre de colonnes
    

```
' ORDER BY 1--
' ORDER BY 2--
```

2. Trouver colonnes affichées
    

```
' UNION SELECT null, null--
```

3. Injecter données utiles
    

```
' UNION SELECT username, password FROM users--
```

---

## 🧠 B. Blind SQLi (pas de réponse visible)

---

### 1. Boolean-based

```
' AND 1=1--
' AND 1=2--
```

👉 comparer les réponses

---

### 2. Time-based

```
' AND SLEEP(5)--
```

👉 délai = condition vraie

---

### 3. OAST (Out-of-band)

👉 exfiltration via serveur externe (Burp Collaborator)

---

## 🧬 C. Second-order SQLi

👉 Injection stockée puis exécutée plus tard

### 🔁 Flow :

1. Injection (stockage)
    
2. Donnée sauvegardée
    
3. Réutilisation vulnérable
    
4. 💥 Injection
    

👉 important :

- pas immédiat
    
- dépend du workflow
    

---

# 🧠 Analogie clé

👉 Payload = **charge dormante**  
👉 Activation = **mauvaise réutilisation backend**

---

# ⚙️ 4. Comprendre UNION SELECT

👉 Sert à **fusionner les résultats de deux requêtes**

```
SELECT a FROM table1
UNION
SELECT b FROM table2
```

👉 utilisé pour afficher des données sensibles dans la réponse

---

# 🧠 5. Mindset Hunter

## 🔥 Questions à se poser :

### 1. Où je peux injecter ?

- champs input
    
- paramètres GET/POST
    
- headers
    
- JSON
    

---

### 2. Où la donnée va ?

- DB
    
- logs
    
- API interne
    
- admin panel
    
- exports
    

---

### 3. Où elle est réutilisée ?

👉 clé pour second-order

---

# 🧪 6. Logique métier & SQLi

👉 SQLi peut :

- bypass restrictions
    
- modifier prix
    
- contourner coupons
    
- accéder à d’autres comptes
    

---

# 🔒 7. Prévention des SQLi

---

## ❌ Mauvais (vulnérable)

```
"SELECT * FROM products WHERE category = '" + input + "'"
```

---

## ✅ Bon (sécurisé)

```
PreparedStatement stmt = connection.prepareStatement(
  "SELECT * FROM products WHERE category = ?"
);
stmt.setString(1, input);
```

---

## 🧠 Pourquoi ça marche ?

👉 séparation :

- SQL (fixe)
    
- data (valeur)
    

👉 impossible de modifier la requête

---

# ⚠️ Limites

Ne protège PAS :

- noms de table
    
- ORDER BY dynamique
    
- colonnes dynamiques
    

👉 solution : whitelist stricte

---

# 🔥 8. Règle d’or

> Une donnée utilisateur ne doit jamais modifier la structure d’une requête SQL

---

# ⚡ 9. Résumé final

- SQLi = injection dans la requête
    
- UNION = extraction de données
    
- Blind = inférence
    
- Second-order = payload dormante
    
- Protection = requêtes paramétrées
    

---

# 🧠 Mentalité élite

👉 Ne cherche pas juste à injecter

👉 Cherche :

- où la donnée circule
    
- où elle est réutilisée
    
- où la logique casse
    

---

# 🚀 Next Step

- API testing (JSON injection)
    
- Access control + SQLi combiné
    
- Business logic + SQLi
    
- Race conditions + SQLi
    

---
