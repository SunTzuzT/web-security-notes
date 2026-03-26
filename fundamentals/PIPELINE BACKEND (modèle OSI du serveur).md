
**But :** visualiser _où_ un contrôle doit exister, et _quoi tester_.

**Pipeline attendu :**

1. **Auth** (qui es-tu ? token/cookie)
    
2. **CheckRole / Authorization** (as-tu le droit ?)
    
3. **CheckOwnership / Tenant** (est-ce à toi ?)
    
4. **ValidateInput** (types/bornes/champs interdits)
    
5. **BusinessLogic** (règles métier + recalculs)
    
6. **DB / Data access** (lecture/écriture)
    
7. **Response** (pas de fuite / pas d’infos en trop)
    

**Règle d’or :**

> Un bug = une couche absente, contournable, ou dans le mauvais ordre.

**Question à écrire :**

> “Quelle couche est la plus fragile ici, et comment je la teste en 1 modification ?”
