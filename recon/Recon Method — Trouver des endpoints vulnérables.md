
## Objectif

Avant d'exploiter une faille, il faut **trouver les endroits où tester**.

La recon consiste à identifier :

endpoints  
paramètres  
API  
flux de données

Ce sont ces éléments qui peuvent contenir :

SQLi  
IDOR  
SSRF  
Business logic

---

# 🧠 Analogie (entrepôt)

Imagine un immense entrepôt.

Avant de voler quelque chose, tu dois identifier :

les portes  
les fenêtres  
les caméras  
les passages

Ces points d’entrée sont les **endpoints**.

---

# 🧭 Méthode en 4 cercles

C’est une méthode utilisée par beaucoup de bug bounty hunters.

surface officielle  
↓  
recon passive  
↓  
surface exposée  
↓  
failles exploitables

---

# 1️⃣ Surface officielle

Observer l’application comme un utilisateur.

Chercher :

pages  
paramètres URL  
formulaires  
API calls

Exemples :

/products?id=10  
/login  
/api/user  
/cart/add

---

# 2️⃣ Recon passive

Identifier les **sous-domaines et services**.

Sources :

certificate transparency  
dns enumeration  
github  
archives web

Exemples :

api.site.com  
admin.site.com  
dev.site.com

Ces sous-domaines contiennent souvent :

versions de test  
API internes  
interfaces admin

---

# 3️⃣ Surface exposée

Observer les **requêtes envoyées au serveur**.

Utiliser :

Burp Proxy

Regarder :

GET  
POST  
cookies  
headers  
JSON

Exemple :

GET /api/products?id=5

ou

{  
 "userId": 12  
}

---

# 4️⃣ Identifier les points sensibles

Certains paramètres sont plus intéressants.

Chercher :

id  
userId  
orderId  
productId  
price  
role  
token

Exemple :

/order?id=1001

Ces paramètres peuvent mener à :

IDOR  
SQLi  
logic bugs

---

# 🔍 Types d'endpoints intéressants

|endpoint|pourquoi|
|---|---|
|search|souvent SQL|
|login|auth bypass|
|cart|logique métier|
|profile|IDOR|
|admin|privilèges|

---

# 🧪 Indices de SQL Injection

Paramètres typiques :

id  
search  
filter  
query  
sort

Exemple :

/search?q=laptop

Test simple :

'

ou

' OR 1=1--

---

# 🔑 Indices d’API

Beaucoup de bugs sont dans les API.

Endpoints typiques :

/api/users  
/api/orders  
/api/products

Les API utilisent souvent :

JSON  
tokens  
IDs

---

# ⚠️ Zones très vulnérables

Les hunters regardent en priorité :

admin panels  
internal APIs  
checkout flows  
file uploads

---

# 🧰 Outils principaux

|outil|rôle|
|---|---|
|Burp Suite|intercepter requêtes|
|Subfinder|trouver sous-domaines|
|Amass|enum DNS|
|Wayback|endpoints historiques|

---

# 🧠 Mentalité hacker

Toujours se demander :

où l'application fait confiance à l'utilisateur ?

Exemples :

IDs dans URL  
paramètres cachés  
cookies

---

# 📌 Règle importante

La majorité des bugs viennent de :

paramètres manipulables

Donc toujours tester :

modification des valeurs

---

# 🧠 Processus complet

Recon  
↓  
Cartographie de l'application  
↓  
Identifier les paramètres  
↓  
Tester les failles  
↓  
Exploiter

---

# 🎯 Ce que cherchent les hunters

Ils cherchent des endroits où l'application :

fait confiance aux données utilisateur

Par exemple :

/api/order?userId=10

Changer :

userId=11

peut révéler :

IDOR

---

# Conclusion

La recon consiste à :

comprendre comment l'application fonctionne

Plus tu comprends le système, plus il devient facile de trouver :

failles logiques  
SQLi  
IDOR
