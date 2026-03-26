
## 🟢 1️⃣ Identification

**Rubrique :** Account  
**Fonction :** Upload avatar  
**Endpoint :** `POST /my-account/avatar`  
**Méthode :** POST  
**Type :** multipart/form-data

---

## 🧱 2️⃣ Structure logique

**Objet :** Avatar (fichier)  
**Action :** Upload  
**ID :** filename (contrôlé par le client)

---

## 🎯 3️⃣ Promesse backend attendue

Le serveur doit garantir :

- Seuls des fichiers image valides sont acceptés
    
- Le fichier n’est jamais exécutable
    
- Le nom du fichier n’est pas contrôlé par le client
    
- Aucun code serveur ne peut être exécuté via l’avatar
    

---

## 🧠 4️⃣ Pipeline backend attendu

Auth  
→ Validate file type réel  
→ Bloquer extensions dangereuses  
→ Renommer côté serveur  
→ Stocker dans dossier non-exécutable  
→ Servir comme contenu statique

---

## 🔬 5️⃣ Ce qui est testé

Modification dans Burp :

filename="myexploit.php"

Ajout dans le body :

<?php echo file_get_contents('/home/carlos/secret'); ?>

---

## 🔎 6️⃣ Résultat observé

- Upload accepté
    
- Fichier stocké dans `/files/avatars/`
    
- Fichier accessible publiquement
    
- Code PHP exécuté
    

Réponse :

Contenu du fichier secret affiché.

---

## 🔴 7️⃣ Décision cassée

Le serveur :

- Ne valide pas réellement le type de fichier ❌
    
- Fait confiance au filename envoyé ❌
    
- Stocke dans un dossier exécutable ❌
    

Type de bug :

File upload → Remote Code Execution

Mais structurellement :

Validation + Politique de stockage cassées.

---

## 🧠 8️⃣ Couche du pipeline touchée

Principalement :

ValidateInput ❌

- Mauvaise configuration de serving ❌
    

---

## 🧩 9️⃣ Mapping avec ta méthode

|Élément|Application|
|---|---|
|Objet|Avatar|
|Action|Upload|
|ID|filename|
|Décision|Fichier sûr uniquement|
|Pipeline cassé|Validation|
|Type de bug|RCE via upload|

---

## 🔥 10️⃣ Leçon importante

Le bug ne vient pas d’un payload complexe.  
Il vient de :

> Confiance excessive au client.

Même pattern que :

- role=admin accepté
    
- price=0 accepté
    
- orderId modifiable
    

---

## 🧭 11️⃣ Si j’étais développeur (mesures correctives)

1. Vérifier le contenu réel (magic bytes + parser image)
    
2. Renommer côté serveur (UUID + extension forcée)
    
3. Stocker dans dossier non exécutable
    

---

# 🧠 Phrase à retenir

Même une RCE est une décision backend mal protégée.
