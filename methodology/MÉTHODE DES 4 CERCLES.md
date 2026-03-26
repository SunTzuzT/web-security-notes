
# 🔵 CERCLE 1 — Surface officielle

👉 _Ce que l’entreprise montre volontairement_

### 🎯 Objectif :

Comprendre le système avant d’attaquer.

### 🔎 Tu cherches :

- Les **rubriques** (modules métier)
    
- Les **objets** (User, Order, Invoice…)
    
- Les **actions** (create, update, refund…)
    
- Les **IDs** utilisés
    
- Les **décisions critiques implicites**
    

### 🧠 Tu te poses :

- Quelle est la promesse backend ici ?
    
- Qui devrait pouvoir faire ça ?
    
- Qu’est-ce qui ne devrait jamais être possible ?
    

### ❌ Tu ne fais pas :

- Scan
    
- Bruteforce
    
- Tests agressifs
    

👉 Tu cartographies.

---

# 🟢 CERCLE 2 — Recon passive

👉 _Ce qui existe autour, sans attaquer_

### 🎯 Objectif :

Découvrir la surface réelle.

### 🔎 Tu cherches :

- Sous-domaines (certificats / CT logs)
    
- sitemap.xml / robots.txt
    
- Endpoints dans le JS
    
- IPs exposées (Shodan)
    
- Environnements staging / admin
    

### 🧠 Règle gravée :

Un domaine à la fois.  
On creuse avant de passer au suivant.

👉 Tu collectes de l’intelligence.

---

# 🟡 CERCLE 3 — Surface exposée

👉 _Ce qui est accessible mais peut être mal protégé_

### 🎯 Objectif :

Tester les contrôles backend.

### 🔎 Tu cherches :

- Auth manquante
    
- Mauvaise vérification de rôle
    
- Ownership non vérifiée (IDOR)
    
- Validation absente
    
- Mauvais recalcul
    
- Mauvaise gestion d’état
    

### 🧠 Pipeline mental :

Auth  
→ CheckRole  
→ CheckOwnership  
→ ValidateInput  
→ BusinessLogic  
→ DB  
→ Response

👉 Tu testes une couche à la fois.

---

# 🔴 CERCLE 4 — Faille réelle

👉 _Ce qui ne devrait jamais être possible_

### 🎯 Objectif :

Démontrer clairement un bug exploitable.

### 🔎 Tu cherches :

- Accès non autorisé
    
- Modification illégitime
    
- Escalade de privilège
    
- Double action
    
- Bypass workflow
    
- Manipulation financière
    

### 🧠 Règle :

Une hypothèse validée =  
“Le serveur accepte ce qu’il devrait refuser.”

---

# 🧠 Ce qu’on cherche à chaque cercle (résumé ultra simple)

|Cercle|Tu cherches quoi ?|
|---|---|
|1|Comprendre les décisions|
|2|Découvrir la surface cachée|
|3|Tester les contrôles|
|4|Prouver une incohérence exploitable|

---

# 🔥 Le fil conducteur

À tous les niveaux, tu cherches :

> Où le serveur fait confiance à la mauvaise chose ?

- Confiance au client ?
    
- Confiance à l’ID ?
    
- Confiance au prix ?
    
- Confiance à l’ordre des étapes ?
    
- Confiance à l’état précédent ?
    

---

# 🧭 Phrase finale à garder

Les applications sont des machines à décisions.  
Les bugs sont des décisions mal protégées.
