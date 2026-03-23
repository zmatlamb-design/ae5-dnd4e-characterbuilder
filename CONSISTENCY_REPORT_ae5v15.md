# Rapport de Cohérence — AE5 Character Sheet v15
**Date :** 2026-03-22
**Analysé par :** Claude (automatique — session pendant absence utilisateur)
**Scope :** FEAT_LIST (60 dons), CLASS_DATA (25 classes), pouvoirs (1 176 entrées nommées)

---

## 1. PROBLÈMES DANS FEAT_LIST

### 🔴 Critique

**1. `Focus de Compétence` — `skill:*` +3 trop large**
- Implémentation actuelle : `bonus:[{type:'skill:*', val:3}]` → +3 à TOUTES les compétences
- PHB1 réel : *Skill Focus* donne +3 à **une seule** compétence au choix
- Impact : chaque fois qu'un joueur équipe ce don, toutes ses 20 compétences gagnent +3 — largement surpuissant
- **Fix recommandé :** Changer en `skill:*` avec un sélecteur de compétence individuel, ou le scinder en don par compétence (comme les autres dons de compétence). Alternative rapide : retirer `skill:*` et mettre `desc` plus précise + laisser le joueur appliquer le +3 manuellement.

**2. `Expertise de Combat` & `Expertise au Combat` — bonus d'attaque incorrect**
- Les deux donnent `{type:'atk', val:1}` = +1 à l'attaque
- PHB1 *Combat Expertise* (p.196) : donne un **bonus à la CA (+1/+2/+3 selon le palier)** lors d'une attaque de mêlée, en échange de –1 à l'attaque
- En 4e, le don qui donne +1/+2/+3 à l'attaque est **Weapon Expertise** (= *Expertise Armée* dans le code)
- Les deux dons se retrouvent ainsi à donner le même effet : doublon + erreur de règle
- **Fix recommandé :** `Expertise de Combat` → `{type:'ca', val:1, cond:'attaque de mêlée'}` ; supprimer `Expertise au Combat` ou le convertir en un don de palier Héroïque distinct

**3. `Action Héroïque` — prérequis `niv.11` erroné**
- Implémentation : `prereq:'niv.11'` → restreint au palier Paragonien
- PHB1 *Heroic Effort* (don humain racial, p.198) : disponible dès le niveau 1 sans condition de niveau
- **Fix recommandé :** `prereq:'Humain'` (c'est un don racial humain, pas un don de palier)

---

### 🟡 Significatif

**4. `Robustesse` / `Robustesse II` / `Robustesse III` — trois dons au lieu d'un**
- PHB1 *Toughness* est un seul don qui monte automatiquement :
  - Palier Héroïque (niv.1–10) : +5 PV
  - Palier Paragonien (niv.11–20) : +10 PV
  - Palier Épique (niv.21–30) : +15 PV
- Avoir trois dons séparés oblige le joueur à en équiper un à chaque palier → non-standard
- **Fix recommandé :** Garder uniquement `Robustesse` avec `bonus:[{type:'hp', val:5}]` et une note dans `desc` sur la progression. Les valeurs II et III pourraient être gérées automatiquement par le niveau.

**5. `Pied Léger Halfelin` — bonus CA vs AO sous-évalué**
- Implémentation : +1 CA contre les attaques d'opportunité
- PHB1 *Nimble Reaction* (don halfelin, p.200) : **+2 CA** contre les attaques d'opportunité
- **Fix recommandé :** `val:2` au lieu de `val:1`

**6. `Résistance Inflexible` — +1 aux JS trop bas**
- Implémentation : `{type:'save', val:1}`
- En 4e, les dons de bonus aux jets de sauvegarde donnent généralement +2 (ex. *Indomitable Will* = +2 vs Vol)
- **Fix recommandé :** `val:2` pour rester dans la norme 4e

---

### 🟢 Mineur / À noter

**7. `Connaissance des Donjons` → mauvaise compétence cible**
- Le don donne `skill:Exploration` qui est mappé à la clé `'rue'` (Rue/Débrouillardise)
- *Dungeoneering* en 4e est une compétence basée sur INT — distinct de Streetwise (CHA)
- La compétence Exploration/Dungeoneering n'existe pas dans le SKILLS array actuel
- **Fix recommandé :** Soit ajouter `{key:'explo', label:'Exploration', stat:'INT'}` dans SKILLS, soit ignorer le bonus en mappant à `null`

**8. `Persuasion Naturelle` → label incohérent**
- Le don est nommé "Persuasion" mais la compétence correspondante s'appelle "Diplomatie" dans SKILLS
- Le mapping `SKILL_LABEL_TO_KEY['Persuasion']='diplomatie'` fonctionne mais la nomenclature est confuse
- **Fix recommandé :** Renommer le don en "Orateur Naturel" ou aligner les noms

---

## 2. PROBLÈMES DANS LES POUVOIRS

### 🔴 Critique

**178 textes d'effets avec artefacts OCR (concaténation multi-pouvoirs)**

L'extraction depuis le PDF PHB a produit des textes d'effet où plusieurs pouvoirs consécutifs du livre ont été fusionnés en un seul champ `effect:`.

Exemples identifiés :
- `Frappe Sacrée` (Paladin) — l'effet contient les textes de "Radiant Smite", "Shielding Smite", "Valiant Strike" et "On Pain of Death" en suite
- `Châtiment Vivifiant` (Paladin niv.3) — contient une référence à "R A V E N M I M U R A Daily" (nom de pouvoir OCR fragmenté)
- `Lumière Ardente` / `Seal of Warding` / `Thunderous Word` (Clerc) — effets combinant plusieurs pouvoirs
- Nombreux pouvoirs de Rôdeur (`Split the Sky`, `Silverstep`, `Unyielding Avalanche`) avec textes concaténés

**Impact :** Lors de l'affichage des fiches de pouvoirs dans l'app, les joueurs voient du texte qui mélange plusieurs effets différents — ambigu et potentiellement induisant en erreur.

**Fix recommandé :** Correction manuelle des 178 effets affectés (filtrer sur les mots-clés `Standard Action`, `Encounter ✦`, `Daily ✦`, `PALADIN`, `FIGHTER`, etc. dans les champs effect). Travail important mais essentiel pour la qualité.

---

### 🟡 Significatif

**Clerc : `Cause Fear` au lieu de `Marque Juste` (Righteous Brand)**
- Le 3ème at-will du Clerc est nommé "Cause Fear" référencé à PHB1 p.66
- Les stats de ce pouvoir (vs Vol, radiant, buff allié) ne correspondent pas à "Cause Fear" (qui serait psychique avec état de peur)
- PHB1 Clerc at-wills standards : **Sacred Flame** (Flamme Sacrée ✅) + **Righteous Brand** (Marque Juste) + **Healing Strike** (Frappe Guérisseuse ✅ si présent)
- La "Marque Juste" (Righteous Brand) donne : FOR vs CA, 1[W]+FOR, l'allié adjacent gagne +1 aux attaques
- **Fix recommandé :** Renommer "Cause Fear" → "Marque Juste" et corriger l'effet selon PHB1 p.65

---

### 🟢 Mineur / À noter

**~1 609 pouvoirs en anglais non traduits**
- De nombreux pouvoirs restent dans leur nom anglais original (ex: "Accelerating Strike", "Acid Storm", etc.)
- Il s'agit principalement de pouvoirs de classes de supplément (PHB2, PHB3, APG)
- Ce n'est pas une erreur de règle mais un travail de traduction en cours

**Thunderwave (Vague de Tonnerre) du Magicien : ✅ correct**
- PHB1 p.163 Thunderwave : Close blast 3, +½NIV+INT vs Vig, 1d6+INT tonnerre, pousse = mod SAG
- L'implémentation est conforme

---

## 3. RÉSUMÉ PRIORITÉS

| Priorité | Issue | Fix estimé |
|----------|-------|------------|
| 🔴 P1 | 178 effets de pouvoirs OCR concaténés | Correction manuelle/semi-auto des 178 entrées |
| 🔴 P1 | `Focus de Compétence` skill:* +3 à tout | Changer comportement → +3 à 1 compétence |
| 🔴 P1 | `Expertise de Combat` donne atk au lieu CA | Corriger bonus type: 'ca' + supprimer doublon |
| 🔴 P1 | `Action Héroïque` prereq niv.11 incorrect | Changer prereq à 'Humain' |
| 🟡 P2 | Clerc at-will "Cause Fear" → "Marque Juste" | Renommer + corriger effet |
| 🟡 P2 | `Robustesse` x3 feats au lieu d'un scalant | Fusionner en 1 don auto-scalant |
| 🟡 P2 | `Pied Léger Halfelin` +1 devrait être +2 | `val:2` |
| 🟡 P2 | `Résistance Inflexible` +1 devrait être +2 | `val:2` |
| 🟢 P3 | `Connaissance des Donjons` mappe à 'rue' | Ajouter clé 'explo' dans SKILLS |
| 🟢 P3 | `Persuasion Naturelle` nom incohérent | Renommer don ou aligner label |
| 🟢 P3 | ~1 609 pouvoirs en anglais | Traduction continue (cosmétique) |

---

## 4. CE QUI EST CORRECT ✅

- Stats de HP/Surges/Défenses pour les 25 classes : conformes PHB1/PHB2/PHB3
- At-wills du Magicien (Cloud of Daggers, Magic Missile, Ray of Frost, Thunderwave) : ✅
- At-wills du Guerrier (Coup Fendu, Frappe Mortissante, etc.) : ✅ (nombreux présents)
- `Grande Vigueur` +2 Vig, `Réflexes Foudroyants` +2 Réf, `Volonté de Fer` +2 Vol : ✅
- `Initiative Améliorée` +4 init : ✅
- `Attaque Puissante` +2 dégâts mêlée : ✅
- Boucliers : +1 CA léger, +2 CA lourd : ✅
- Sprint 1A/1B (bonus dons→compétences, auto-calc attaque) : ✅ livré en v15
- RACE_DATA (vitesses Elfe:7, Nain:5, etc.) : à vérifier manuellement (données non accessibles via le parser utilisé)

---

*Ce rapport a été généré automatiquement par analyse statique du code source. Une révision manuelle avec le PHB1 ouvert est recommandée avant correction des issues P1.*
