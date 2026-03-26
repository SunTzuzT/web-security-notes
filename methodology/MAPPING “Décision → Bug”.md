
- **Ownership cassée** → IDOR / BAC objet
    
- **Rôle mal vérifié** → Broken Access Control (admin/user)
    
- **Valeur sensible acceptée** (price/total/discount) → Business Logic / Price tampering
    
- **État / workflow mal géré** (refund, cancel, approve) → State machine flaw
    
- **Champs inattendus acceptés** (`role`, `isAdmin`, `status`) → Mass Assignment
    
- **Requête rejouable** (double action) → Idempotence flaw / Double spend
    
- **Deux requêtes concurrentes** → Race condition
    
- **Validation absente** (type/bornes) → Injection/Tampering (selon contexte)
    

Phrase à garder :

> “Je cherche quelle promesse est cassée → le type de bug tombe tout seul.”
