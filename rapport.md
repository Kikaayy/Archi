# Rapport du Projet y86 - Sujet 4

**Nom** : COUSIN Kellian
**Date** : 4 mai 2025

---

## Introduction
Ce projet vise à étendre le simulateur y86 en deux étapes clés :  
1. **Factorisation** des instructions `rrmovl` et `irmovl` pour libérer des opcodes.  
2. **Ajout** de nouvelles instructions (`incl`, `decl`, `loop`, `loope`, `loopne`) et implémentation de fonctions avancées (`strncpy`).  
---
## **Ci-joint avec ce fichier les "saves" noté tel eXqY.json (X = Exercice, Y = Question) pour chaque question**
---


## Exercice 1 : Factorisation de `rrmovl` avec `irmovl`

### Question 1
- Fusion des instructions `rrmovl` et `irmovl` sous le même `icode` RIMOVL
- Utilisation du champ `ifun` pour distinguer les cas :
  - `ifun=0` : Accès à `valA` (registre → registre)
  - `ifun=1` : Accès à `valC` (immédiat → registre)
- Modification de l'encodage : `irmovl 3 0 valC,rB => irmovl 2 1 valC,rB`

### Question 2
- Ajout de tests critiques dans test.ys :
  - Transfert entre registres avec valeurs négatives
  - Chargement d'une constante maximale (0xFFFFFFFF)
- Mise à jour du HCL pour gérer les deux cas via RIMOVL

## Exercice 2 : Ajout des instructions `incl` et `decl`

### Question 1
- Ajout de l'instruction `incl` dans le jeu d'instructions
- Mise à jour du fichier HCL :
  - Ajout du symbole `INCL`
  - Modification des signaux de contrôle pour inclure `INCL`
  - Mise à jour des conditions pour les registres et l'ALU
- Création d'un nouveau fichier de test `test.ys` pour tester l'instruction `incl`
- Tests de l'instruction sur différents registres avec différentes valeurs initiales

### Question 2
- Choix de `rA` comme registre de destination pour `incl`
- Ajout de l'instruction dans le HCL :
  ```hcl
  intsig INCL 'instructionSet.get("incl").icode'
  ```
- Mise à jour des signaux de contrôle :
  - `need_regids` : Ajout de INCL
  - `instr_valid` : Ajout de INCL
  - `srcA` : Ajout de INCL
  - `dstE` : Ajout de INCL
  - `aluA` : Ajout de INCL
  - `aluB` : Ajout de INCL
  - `alufun` : Ajout de INCL
  - `set_cc` : Ajout de INCL

**Pourquoi avoir choisi `rA` au lieu de `rB` pour `incl` ?**  
- **Cohérence avec `iaddl`** : L’instruction `iaddl` utilise `rB` comme registre de destination.  
- **Optimisation du décodage** : En utilisant `rA`, on évite de modifier le champ `rB` dans le byte de registres, ce qui simplifie le traitement dans l’étage *Decode*.  
- **Exemple** :  
  - `incl %eax` → `rA = %eax`, `rB` ignoré.  
  - Pas de conflit avec les autres instructions utilisant `rB` (ex: `rmmovl`).

### Question 3
- Factorisation de `decl` avec `incl` via le champ `ifun` :
  - `ifun=0` : Addition de +1 (incl)
  - `ifun=1` : Addition de -1 (decl)
- Réutilisation du circuit ALU existant
**Comment factoriser `decl` avec `incl` dans le HCL ?**  
- **Réutilisation de l’ALU** :  
  - `incl` → Addition de `+1`.  
  - `decl` → Addition de `-1` (via `ifun=1`).  
- **Aucun signal supplémentaire** : Le calcul de `valE` reste identique, seul `aluB` change.  
- **Avantage** : Évite la duplication de code HCL pour des opérations symétriques.

### Question 4
**VOIR ANNEXE e2q4.json (code pour dedans)**

## Exercice 3 : Ajout de l'instruction `loop`

### Question 1
- Ajout d'un `icode` dédié pour `loop`
- Décrémentation implicite de `%ecx`
- Branchement conditionnel sans affecter les codes de condition

### Question 2
- Préservation des codes de condition pour les instructions suivantes
- Les instructions `loope`/`loopne` dépendent de `ZF` actuel

## Exercice 4 : Ajout des instructions `loope`/`loopne`

### Question 1
- Partage du même `icode` que `loop`
- Utilisation du champ `ifun` pour distinguer les cas :
  - `ifun=0` : loop (branche si `%ecx != 0`)
  - `ifun=1` : loope (branche si `%ecx != 0 && ZF=1`)
  - `ifun=2` : loopne (branche si `%ecx != 0 && ZF=0`)

### Question 2
- Ajout des instructions `loope` et `loopne` dans le HCL
- Modification de la logique de branchement dans `new_pc` :
  ```hcl
  # LOOPE instruction: si ecx != 0 et Z = 1
  icode == LOOP && ifun == 1 && valE != 0 && Bch : valC;
  # LOOPNE instruction: si ecx != 0 et Z = 0
  icode == LOOP && ifun == 2 && valE != 0 && !Bch : valC;
  ```
- Mise à jour du fichier de test pour inclure les tests de `loope` et `loopne`
- Tests des différentes conditions de boucle avec différents états du registre `%ecx` et du flag `ZF` 

### Question 3
- Utilisation de `loopne` dans `strncpy` pour :
  - Combiner la décrémentation de `%ecx` et la vérification de `ZF`
  - Arrêt intelligent si la sentinelle `0` est trouvée ou si `%ecx` atteint `0`
  - Réduction du nombre d'instructions par itération 
  **VOIR ANNEXE e4q3.json (code pour dedans)**