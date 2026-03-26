https://portswigger.net/web-security/learning-paths/server-side-vulnerabilities-apprentice/path-traversal-apprentice/file-path-traversal/lab-simple
# 🧠 Path Traversal — Fiche récap (PortSwigger)

## 🔎 Nom du pattern

**Path Traversal (Directory Traversal)**

---

## 📍 Contexte du lab

- Application web affichant des images
    
- Paramètre contrôlé par le client :
    

image=62.jpeg

- Ce paramètre est utilisé côté serveur pour accéder à un fichier sur le système
    

---

## ⚠️ Hypothèse backend cassée

> _Le serveur suppose que la valeur fournie par l’utilisateur correspond uniquement à un fichier image autorisé dans un répertoire précis._

En réalité :

- l’input utilisateur est **directement utilisé** dans un chemin fichier
    
- aucune normalisation / validation stricte
    
- possibilité de sortir du répertoire prévu
    

---

## 🧪 Exploitation observée

Modification du paramètre :

image=../../../etc/passwd

Résultat :

- le serveur résout le chemin
    
- lit un fichier système arbitraire
    
- le contenu du fichier est bien **récupéré côté backend**
    

👉 Le fichier ciblé contenait notamment **des noms d’utilisateurs**, confirmant l’accès à une ressource sensible.

---

## 🛠️ Outils et méthodes utilisés

### 🔹 Burp Suite — Repeater (point clé)

- Interception de la requête
    
- Envoi dans **Repeater**
    
- Tests successifs avec différentes valeurs de chemin
    

Analyse basée sur :

- **code de réponse HTTP**
    
    - `200 OK` → le fichier est accessible et traité
        
    - `400 Bad Request` → tentative bloquée / invalide
        
    - `403 Forbidden` → contrôle partiel
        
    - `404 Not Found` → chemin non résolu
        

👉 Le **status code** permet de comprendre :

- si la requête passe réellement côté backend
    
- si un filtrage est présent
    
- si le chemin est interprété par le serveur
    

---

### 🔹 Différence navigateur vs Burp (point important)

- Le **navigateur affichait une image**
    
- Les données récupérées (contenu texte du fichier) **n’étaient pas exploitables visuellement via l’interface web**
    
- En revanche, via **Burp**, le contenu brut de la réponse était visible
    

👉 **La faille existe côté backend**, même si :

- le front n’affiche pas correctement les données
    
- le rendu masque l’impact réel
    

📌 **L’exploitabilité ne dépend pas de l’affichage front-end.**

---

## 🔎 Pourquoi Repeater est essentiel

- Permet de tester **sans dépendre du front**
    
- Accès direct aux **réponses brutes**
    
- Observation du contenu réel retourné par le serveur
    
- Comparaison fine des réponses (code, taille, contenu)
    

👉 **Le front n’est jamais une barrière de sécurité.**

---

## 💥 Impact réel (au-delà du POC)

`/etc/passwd` est un **POC**, pas l’objectif final.

Impacts réels possibles :

- récupération de noms d’utilisateurs
    
- lecture de fichiers de configuration
    
- accès à des secrets (.env, clés API)
    
- lecture de logs applicatifs
    
- préparation d’attaques plus graves (LFI → RCE)
    

---

## 🌍 Où ce pattern apparaît en vrai

- systèmes de téléchargement
    
- affichage de documents
    
- exports PDF / CSV
    
- pièces jointes
    
- endpoints API de fichiers
    
- applications legacy
    

👉 Très fréquent en **pentest web** et **bug bounty**.

---

## ❌ Erreur classique à éviter

Penser que :

> _“La faille n’est pas grave car le navigateur n’affiche rien d’exploitable.”_

❌ Faux.  
👉 **La sécurité se juge côté backend, pas côté rendu UI.**

---

## ✅ Compréhension clé (à retenir)

> **Le path traversal n’est pas une question de `../`  
> mais une question de confiance backend dans un input utilisateur.**

Dès qu’un input devient :

- un chemin
    
- une ressource interne  
    → **test immédiat**.
    

---

## 🎯 Question réflexe à se poser

> _“Est-ce que cette valeur est utilisée côté serveur pour accéder au système de fichiers ?”_

Si oui → **Path Traversal potentiel**.

---

## 📌 Résumé en une phrase

> **Le path traversal permet l’accès à des fichiers arbitraires lorsque le backend utilise un input utilisateur comme chemin sans le sécuriser correctement ; même si le navigateur ne rend pas les données exploitables, Burp permet d’observer le contenu réel retourné par le serveur (codes HTTP 200 / 400 / 403).**
> 