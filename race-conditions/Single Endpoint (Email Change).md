---  
  
## 🔍 1. Mécanisme backend  
  
### Flow réel (supposé)  
1. pending_email = X (DB)  
2. génération token  
3. préparation email  
4. envoi email (async / différé)  
  
### ⚠️ Problème  
- `pending_email` = **unique**  
- `pending_email` = **mutable**  
  
➡️ état partagé écrasable  
  
---  
  
## 🧠 2. Intuition clé  
  
> Une seule case mémoire écrasée par plusieurs requêtes  
  
### 🎯 Analogie (tableau blanc)  
- A écrit une adresse  
- B écrit par-dessus  
- le facteur lit après  
  
➡️ email envoyé au mauvais destinataire  
  
---  
  
## ⚠️ 3. Fail structurel  
  
### 🔍 Source de vérité  
- DB → `pending_email`  
  
### 💣 Problème  
- utilisée à plusieurs moments différents  
  
### ❌ Désynchronisation  
- token généré avec état A  
- email envoyé avec état B  
  
➡️ TOCTOU interne  
  
---  
  
## 🧠 Analogie (restaurant)  
1. serveur prend commande (A)  
2. ticket modifié (B)  
3. cuisine exécute ticket modifié  
  
➡️ mauvais plat au bon client  
  
---  
  
## 🔎 4. Évaluation du comportement  
  
### 🔍 Signal clé  
> premier lien devient invalide  
  
### 🧠 Déduction  
- 1 seul état stocké  
- pas de versioning  
- overwrite direct  
  
➡️ surface idéale pour race  
  
---  
  
## 🧪 5. Indices exploitables  
  
### 🔍 Test  
- envoyer plusieurs changements rapidement  
  
### 💥 Signal critique  
- email reçu ≠ email demandé  
  
➡️ preuve :  
- lecture DB ≠ écriture initiale  
  
---  
  
## 💣 6. Exploit logique  
  
### 🎯 Objectif  
Créer une collision contrôlée  
  
| Requête | Action |  
|--------|--------|  
| A | pending_email = attacker |  
| B | pending_email = victim |  
| A | envoi email MAIS lit état B |  
  
---  
  
### 🧠 Résultat  
- mail reçu chez attacker  
- contenu = victim  
  
➡️ token valide pour victim  
  
---  
  
## ⚡ 7. Tokens invalides (explication)  
  
### 🔍 Observation  
> token écrasé  
  
### 🧠 Cause  
- nouvel état → invalide ancien token  
- système = **single state**  
  
---  
  
### 🔁 TOCTOU ici  
- génération token  
- stockage état  
- validation finale  
  
---  
  
## 🧬 8. Single vs Multi Endpoint  
  
### 🔍 Multi-endpoint  
- plusieurs routes  
- ex : /apply-coupon → /checkout  
➡️ race logique métier  
  
---  
  
### 🔍 Single-endpoint  
- même route (`/change-email`)  
- requêtes identiques  
- payload différent  
  
➡️ race sur **état interne**  
  
---  
  
### 🧠 Différence clé  
  
| Type | Cible | Visibilité |  
|------|------|-----------|  
| Multi-endpoint | logique métier | visible |  
| Single-endpoint | état interne | caché |  
  
---  
  
## 🔑 9. Insight critique  
  
> Même endpoint ≠ même effet backend  
  
Pourquoi :  
- état partagé (DB / session)  
- traitement non atomique  
- timing non contrôlé  
  
---  
  
## 🧠 10. Pattern mental (hunter)  
  
Toujours chercher :  
  
- état partagé  
- écriture non atomique  
- lecture différée  
- overwrite sans historique  
  
---  
  
### ❓ Question clé à se poser  
  
> "Et si une autre requête modifie l’état entre temps ?"  
  
---  
  
## 🧪 11. Signaux terrain  
  
- tokens invalidés rapidement  
- incohérence email / action  
- effets non déterministes  
- résultats différents à répétition  
  
---  
  
## 🎯 TL;DR  
  
- une seule source de vérité  
- modifiée par plusieurs requêtes  
- utilisée à plusieurs moments  
  
➡️ collision possible = exploit  
  
---
