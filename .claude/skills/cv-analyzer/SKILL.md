---
name: cv-analyzer
description: Analyser un CV et identifier les points positifs et négatifs par rapport à un poste cible. Utiliser quand l'utilisateur demande d'analyser un CV, évaluer une candidature, ou reviewer un curriculum vitae.
argument-hint: [chemin-vers-cv]
---

# Analyseur de CV

Analyse un CV par rapport à un poste cible et produit un rapport structuré des points forts et points faibles.

## Processus

1. **Collecter les informations**
   - Si le CV n'est pas fourni en argument, demander le chemin du fichier
   - Demander le poste cible visé (intitulé + contexte si disponible)

2. **Lire et analyser le CV**
   - Lire le fichier CV (PDF, DOCX, ou texte)
   - Extraire les informations clés: expériences, compétences, formation, langues, certifications

3. **Évaluer par rapport au poste**
   - Comparer les compétences du candidat avec les exigences typiques du poste
   - Identifier l'adéquation entre l'expérience et le niveau attendu

4. **Produire le rapport**

## Format du rapport

```markdown
## Analyse du CV pour le poste de [POSTE CIBLE]

### Points Positifs
- [Point fort 1 avec justification]
- [Point fort 2 avec justification]
- ...

### Points à Améliorer
- [Point faible 1 avec suggestion d'amélioration]
- [Point faible 2 avec suggestion d'amélioration]
- ...

### Adéquation globale
[Évaluation synthétique de l'adéquation candidat/poste]

### Recommandations
- [Actions concrètes pour améliorer le CV]
```

## Critères d'évaluation

### Points positifs potentiels
- Expérience directement pertinente pour le poste
- Compétences techniques recherchées
- Progression de carrière cohérente
- Formations et certifications valorisantes
- Réalisations quantifiées et mesurables
- Mots-clés alignés avec le secteur
- Structure claire et lisible
- Langues maîtrisées pertinentes

### Points négatifs potentiels
- Lacunes de compétences clés pour le poste
- Trous inexpliqués dans le parcours
- Expérience insuffisante ou non pertinente
- Manque de réalisations concrètes
- Présentation confuse ou surchargée
- Fautes d'orthographe ou incohérences
- Informations obsolètes ou non pertinentes
- Absence de personnalisation pour le poste
