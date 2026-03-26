
# 🔥 1. Qu’est-ce qu’une documentation machine ?

> Documentation structurée lisible par des outils (JSON / YAML)

Formats courants :

- OpenAPI / Swagger (`.json` / `.yaml`)
    
- `/openapi.json`
    
- `/swagger.json`
    

---

# ⚡ 2. Pourquoi c’est ULTRA puissant

👉 Parce que ça te donne :

- tous les endpoints
    
- toutes les méthodes (GET, POST…)
    
- tous les paramètres
    
- les schémas JSON attendus
    
- parfois les rôles / permissions
    

---

## 💥 Traduction bug bounty

> **Tu vois directement toute la surface d’attaque**

---

# 🔍 3. Où trouver cette documentation

Endpoints classiques :

```
/api
/swagger
/swagger/index.html
/openapi.json
/api-docs
```

---

# 🧠 4. Outils à utiliser

---

## 🔴 Burp Suite

👉 Scanner + exploration

- crawl endpoints
    
- détecte automatiquement la doc
    
- analyse OpenAPI
    

---

## 🟠 Postman

👉 tester les endpoints facilement

- importer OpenAPI
    
- envoyer requêtes rapidement
    
- modifier JSON
    

---

## 🔵 OpenAPI Parser

👉 analyser la doc

- lire structure API
    
- comprendre endpoints
    
- voir paramètres cachés
    

---

## 🟣 SoapUI

👉 alternative à Postman

---

# 🔁 5. Workflow d’un bug bounty hunter

---

## Étape 1 — Trouver la doc

```
Recherche endpoints → /swagger /openapi.json
```

---

## Étape 2 — Lire la doc

👉 Identifier :

- endpoints sensibles
    
- paramètres intéressants
    
- champs cachés
    

---

## Étape 3 — Importer dans Postman / Burp

👉 automatiser les tests

---

## Étape 4 — Tester 🔥

- modifier JSON
    
- changer IDs
    
- ajouter champs
    
- tester autorisations
    

---

# 💥 6. Pourquoi c’est dangereux pour une app

> Une doc exposée = roadmap pour attaquer

---

## Exemple :

Tu vois :

```
DELETE /api/users/{id}
```

👉 tu testes :

- user normal → delete ?
    
- autre ID → IDOR ?
    
- sans auth → accès ?
    

💥 faille possible

---

# 🧠 7. Ce que tu dois chercher

- endpoints admin
    
- endpoints delete / update
    
- paramètres sensibles
    
- champs non documentés
    
- différences entre endpoints
    

---

# ⚠️ 8. Limite importante

> La doc n’est pas toujours complète

👉 Donc :

- tester quand même
    
- bruteforce endpoints
    
- observer trafic réel
    

---

# 🧠 9. Vision mentale

```
Documentation = carte du système
Toi = explorateur
```

---

# ⚡ 10. Phrase clé

> **“Si la documentation est exposée, l’attaque devient beaucoup plus facile.”**

---

# 🚀 11. Conclusion

Utiliser la documentation API permet de :

- gagner du temps
    
- découvrir des endpoints cachés
    
- comprendre la logique backend
    
- tester efficacement
    

---