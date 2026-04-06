

_(Fiche Obsidian prête à copier)_

---

## 🧠 Analogie (VISION HACKER)

Imagine un coffre 🔒 :

- **Force brute** → tu testes toutes les combinaisons
- **Énumération** → tu vérifies d’abord si le coffre existe vraiment
- **Rate limit / lockout** → le garde te ralentit… mais pas intelligemment
- **Credential stuffing** → tu testes des clés déjà volées ailleurs

👉 Le vrai hacker :

> ❌ ne brute force PAS bêtement  
> ✅ réduit l’espace de recherche AVANT d’attaquer

---

## 📌 Définition

### 🔹 Force brute

Tester automatiquement des combinaisons :

username + password

### 🔹 Énumération

Déterminer si un **username est valide** avant brute force

👉 Gain énorme :

1000 users → 50 users valides → attaque ciblée

---

## ⚙️ Fonctionnement réel (backend)

Flow login classique :

1. Vérifie si user existe  
2. Si oui → vérifie password  
3. Retourne réponse

💣 Problème :  
👉 Si ces étapes ne sont pas uniformes → fuite d’info

---

## 💣 Points de cassure (CRITIQUES)

### 1. Énumération username

Différences exploitables :

- 🔢 Status code
- 💬 Message erreur
- ⏱️ Temps réponse

---

### 🔍 Exemple mental

user invalide → réponse immédiate  
user valide → check password → plus lent

👉 Tu peux détecter un user juste avec le **timing**

---

### 2. Messages d’erreur

❌ Mauvais :

User not found  
Password incorrect

✅ Bon :

Invalid credentials

---

### 3. Status code

❌ Mauvais :

200 → user faux  
401 → user correct

👉 jackpot pour enumeration

---

### 4. Timing attack

👉 Très puissant en bug bounty

Technique :

- envoyer mot de passe LONG (ralentit le hash)
- comparer les temps

---

## 🔥 Patterns d’attaque (RÉUTILISABLES)

### 🧪 1. Username enumeration

Checklist :

- tester plusieurs usernames
- observer :
    - length
    - status
    - timing
    - wording

---

### 🧪 2. Password brute force optimisé

❌ Mauvais :

aaaaaa → zzzzzz

✅ Bon :

Password123  
Welcome1!  
Company2024!

👉 basé sur comportement humain

---

### 🧪 3. Password mutation

Exemples :

mypassword  
Mypassword1!  
Mypassword2!

👉 ultra fréquent

---

### 🧪 4. Credential stuffing

Utilise :

email:password leaks

👉 fonctionne car :

> les gens réutilisent leurs creds

---

## 🛡️ Protections & Bypass

---

### 🔒 1. Lockout (verrouillage compte)

👉 après X tentatives → compte bloqué

---

### 💣 Bypass classique

Stratégie :

user1 → pass1  
user2 → pass1  
user3 → pass1

👉 jamais dépasser limite par user

---

### 🔥 Technique clé

max_attempts = 3  
  
→ tester 3 passwords sur 100 users

👉 300 tests sans lock

---

### 🧠 Mentalité :

> Tu attaques LARGE, pas profond

---

### 🔒 2. IP Rate Limit

👉 bloque ton IP

---

### 💣 Bypass :

- changer IP
- X-Forwarded-For spoof
- proxy rotation

---

### 🔥 Autre bypass avancé

👉 plusieurs passwords en 1 requête

---

## ⚠️ Protection mal implémentée

### 💣 Exemple critique :

fail → compteur++  
success → reset compteur

👉 bypass :

test1  
test2  
login legit  
reset

👉 boucle infinie 🔁

---

## 🔐 HTTP Basic Auth

---

### ⚙️ Fonctionnement

Header :

Authorization: Basic base64(username:password)

👉 envoyé à CHAQUE requête

---

### 💣 Failles majeures

- ❌ credentials statiques
- ❌ brute force facile
- ❌ pas de protection native
- ❌ vulnérable MITM (si pas HTTPS)
- ❌ vulnérable CSRF

---

### 🧠 Impact réel

👉 tu récupères creds → réutilisation ailleurs

---

## 🔍 Checklist Bug Bounty (TRÈS IMPORTANT)

Quand tu vois un login :

### 🎯 Phase 1 — Énumération

- différences de réponse ?
- timing différent ?
- message différent ?
- email leak dans réponse ?

---

### 🎯 Phase 2 — Brute smart

- passwords communs
- mutation
- patterns humains

---

### 🎯 Phase 3 — Protections

- lockout ?
- rate limit ?
- captcha ?

---

### 🎯 Phase 4 — Bypass

- test multi-users
- test reset compteur
- test IP spoof

---

## 🧠 Mental Model (CE QUE TU DOIS TE DEMANDER)

- Est-ce que le backend traite différemment :
    - user invalide vs valide ?
- Où est faite la vérification ?
- Est-ce qu’il y a une logique conditionnelle ?
- Est-ce que je peux réduire l’espace de recherche ?
- Est-ce que la protection est globale ou par user ?

---

## ⚡ Résumé brutal

👉 Le vrai skill ici :

- ❌ pas brute force
- ✅ comprendre la logique backend
- ✅ détecter différences invisibles
- ✅ optimiser l’attaque

---

## 🧠 Niveau bug bounty (important pour toi)

👉 Cette catégorie :

- 🔹 facile techniquement
- 🔹 MAIS ultra dépendante du détail

👉 Ce qui fait la diff :

> ta capacité à OBSERVER
