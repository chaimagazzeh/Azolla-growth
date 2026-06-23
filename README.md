# Azolla pinnata — Prédiction de la biomasse (Approche 1)

Pipeline de modélisation prédictive pour le mémoire de master sur la production de biomasse
d'*Azolla pinnata* en conditions contrôlées.

## Objectif

**Approche 1 — pré-expérimentale :** prédire la biomasse finale (M21) d'un bac à partir de ses
conditions de culture initiales (composition du milieu, luminosité, pH). Cible secondaire : le
taux de croissance relatif TCR = ln(M21/M0)/t.

> **Approche 2 (déplétion de la CE)** n'est *pas* implémentée : les mesures de conductivité
> électrique sont mono-temporelles (une valeur par bac, pas de série), ce qui empêche d'ajuster
> une constante de décroissance. Elle est conservée dans le mémoire comme méthodologie proposée.

## Échantillon : 16 bacs pour l'analyse, 24 bacs pour la modélisation

Le mémoire utilise deux périmètres d'échantillon différents selon l'objectif. C'est un choix
méthodologique assumé, pas une incohérence :

- **Analyses statistiques et biochimiques → 16 bacs.** Les dosages biochimiques (chlorophylle,
  caroténoïdes, etc.) n'ont été réalisés que sur 16 bacs. Toute analyse qui croise la biomasse
  avec ces marqueurs est donc restreinte à ce sous-ensemble, faute de mesures sur les autres.
- **Modélisation prédictive → 24 bacs (tous les bacs disposant d'une biomasse finale).** Le modèle
  n'a besoin que des conditions de culture (composition, lumière, pH) et de M21. Ces variables
  sont disponibles pour les 24 bacs. À une taille d'échantillon aussi faible, écarter des
  observations valides dégraderait inutilement la robustesse du modèle et la validation LOOCV.
  On inclut donc **toutes** les données exploitables pour la modélisation.

> Principe : on utilise le périmètre adapté à chaque question — le sous-ensemble mesuré
> biochimiquement (n=16) pour les analyses qui en dépendent, et l'ensemble complet (n=24) pour
> la modélisation qui n'en dépend pas. À déclarer explicitement dans la section Matériels &
> Méthodes du mémoire.

## Structure du dépôt

```
data/
  dataset azolla_v2.xlsx       # données brutes (pH, biomasse). Feuille "Feuil2" = référence.
  medium_composition.csv       # À REMPLIR : composition élémentaire des milieux (voir plus bas)
notebooks/
  approach1_biomass_model.ipynb  # notebook principal, documenté cellule par cellule
outputs/                       # figures + tableaux générés automatiquement
RESULTS_AND_LIMITATIONS.md     # interprétation des résultats et limites
```

## Installation et exécution

```bash
python -m venv .venv && source .venv/bin/activate
pip install pandas numpy matplotlib seaborn scikit-learn xgboost shap openpyxl jupyter
jupyter notebook notebooks/approach1_biomass_model.ipynb
```

Le notebook contient aussi une cellule `%pip install` en tête (section 0). Exécuter les cellules
dans l'ordre. Les figures et tableaux sont écrits dans `outputs/`.

## Le fichier `data/medium_composition.csv`

C'est le **seul fichier à compléter manuellement**. Tant qu'il est vide, le notebook encode le
milieu en variables indicatrices (one-hot) et tourne quand même. Dès qu'il est rempli, le modèle
utilise les vraies concentrations élémentaires — aucune modification de code nécessaire.

### Format attendu

- **Une ligne par milieu** (les noms doivent correspondre exactement à ceux des données :
  `IRR2`, `Yoshida`, `Modified Hoagland`, `NPK`).
- **Une colonne par nutriment.** Les colonnes par défaut sont N, P, K, Ca, Mg, Fe, S — vous
  pouvez en ajouter/retirer librement, le code lit toutes les colonnes sauf `Medium`.
- **Valeurs = concentration de l'élément en mmol/L** dans la solution nutritive préparée.

### Quelles valeurs mettre ?

Les concentrations proviennent de la **recette de préparation** de chaque milieu (composition
théorique connue d'après le protocole / la littérature). Pour chaque sel ajouté, convertir la
masse en mmol/L de l'élément concerné. Exemple de remplissage (valeurs fictives à remplacer) :

| Medium            | N    | P   | K    | Ca  | Mg  | Fe   | S   |
|-------------------|------|-----|------|-----|-----|------|-----|
| IRR2              | 2.0  | 0.1 | 1.0  | 0.5 | 0.4 | 0.02 | 0.4 |
| Yoshida           | 5.7  | 0.3 | 1.0  | 1.0 | 1.6 | 0.04 | 1.6 |
| Modified Hoagland | 0.0  | 1.0 | 6.0  | 4.0 | 2.0 | 0.05 | 2.0 |
| NPK               | 4.0  | 0.5 | 3.0  | 0.0 | 0.0 | 0.00 | 0.0 |

> Une valeur de **0** est valide (élément absent — ex. azote nul dans Modified Hoagland par
> conception). Laisser une cellule **vide** signifie « inconnu » ; si un milieu n'a aucune
> valeur, le notebook repasse en mode one-hot.

## Variables utilisées par le modèle

| Variable     | Source                                  |
|--------------|------------------------------------------|
| Nutriments   | `medium_composition.csv` (ou one-hot)   |
| `light_lux`  | Table du bac (T1=1141, T2=834, T3=727)  |
| `pH_mean`    | Moyenne du pH par bac sur la période    |

**Exclues** (et pourquoi) : humidité et photopériode (constantes → variance nulle), TDS et
salinité (colinéaires à la CE). Détails dans `RESULTS_AND_LIMITATIONS.md`.

## Conception expérimentale

24 bacs, plan **déséquilibré** : 3 NPK, 7 IRR2, 7 Yoshida, 7 Modified Hoagland. Biomasse initiale
standardisée à 36 g ± 2 g. Pas de bac témoin dans ce jeu de données. Identifiant unique de bac =
`Medium + Table + Bac`.