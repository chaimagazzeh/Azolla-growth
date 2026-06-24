# Azolla pinnata — Prédiction de la biomasse (Approche 1)

Pipeline de modélisation prédictive pour le mémoire de master sur la production de biomasse
d'*Azolla pinnata* en conditions contrôlées.

## Objectif

**Approche 1 — pré-expérimentale :** prédire le **TCR** (taux de croissance relatif) d'un bac à
partir de ses conditions de culture initiales (composition élémentaire du milieu, lumière, pH,
biomasse initiale, température, humidité). TCR = (ln M₁ − ln M₀)/(t₁−t₀), Hunt (1982).

> **Approche 2 (déplétion de la CE)** n'est *pas* implémentée : les mesures de conductivité
> électrique sont mono-temporelles (une valeur par bac), ce qui empêche d'ajuster une constante de
> décroissance. Conservée dans le mémoire comme méthodologie proposée.

## Données utilisées

- **`data/dataset azolla_v3.xlsx`, feuille `Feuil2` (3ᵉ feuille)** — source de référence : la plus
  complète (biomasse initiale **et** finale par bac, composition élémentaire, bloc).
- **`data/medium_composition.csv`** — composition élémentaire par milieu (mmol/L), générée depuis la
  v3. Lue automatiquement par le notebook.
- **`data/tmp_and_humidity.txt`** — log du capteur de salle (température, humidité), agrégé par jour.

## Résultats (LOOCV, n=20, plan 6/6/5)

| Modèle | RMSE | MAE |
|---|---|---|
| **Régression linéaire** | 0.00590 | 0.00462 |
| Random Forest | 0.00593 | 0.00436 |
| Ridge | 0.00615 | 0.00465 |
| XGBoost | 0.00977 | 0.00871 |

Baseline (moyenne) ≈ 0.0093. Les modèles simples réduisent l'erreur d'environ un tiers ; la
**régression linéaire** est la meilleure (parcimonie). Au-delà de `M0` (facteur en partie mécanique,
terme du TCR), le **pH** et l'**éclairement** ressortent comme co-déterminants, suivis d'un gradient
nutritif (P, Fe, N…). Détails et caveats dans `RESULTS_AND_LIMITATIONS.md`.

## Échantillon : 16 bacs pour l'analyse, 20 pour la modélisation

- **Analyses biochimiques : 16 bacs** (seuls bacs dosés biochimiquement).
- **Modélisation : 20 bacs (plan le plus équilibré possible).** L'échantillon de 24 bacs est ramené
  à 20 pour cohérence avec le jeu biochimique, en visant l'équilibre : NPK=3 (contrôle) conservé, et
  les 17 bacs restants répartis en **6/6/5** (partition entière la plus équilibrée de 17 ; 6/6/6
  impossible). Règle reproductible (`RANDOM_STATE=42`) : on retire d'abord le bac au pH incomplet
  (`IRR2_Bloc1_B4`), puis on tire au hasard les bacs restants pour atteindre les tailles cibles.
  Effectif final : NPK=3, IRR2=6, Yoshida=6, Modified Hoagland=5.

## Installation et exécution

```bash
python -m venv .venv && source .venv/bin/activate
pip install pandas numpy matplotlib seaborn scikit-learn xgboost shap openpyxl jupyter
jupyter notebook notebooks/approach1_biomass_model.ipynb
```

Le notebook contient aussi une cellule `%pip install` (section 0). Exécuter les cellules dans
l'ordre ; figures et tableaux sont écrits dans `outputs/`.

## Structure du dépôt

```
data/
  dataset azolla_v3.xlsx       # données de référence (feuille Feuil2)
  medium_composition.csv       # composition élémentaire (générée depuis la v3)
  tmp_and_humidity.txt         # log capteur température/humidité
notebooks/
  approach1_biomass_model.ipynb  # notebook principal, documenté cellule par cellule
outputs/                       # figures + tableaux générés
RESULTS_AND_LIMITATIONS.md     # interprétation détaillée et limites
```

## Le fichier `data/medium_composition.csv`

Généré automatiquement depuis la 3ᵉ feuille de la v3 ; modifiable à la main si besoin. Le notebook
le relit tel quel.

- **Une ligne par milieu** (`IRR2`, `Yoshida`, `Modified Hoagland`, `NPK`).
- **Une colonne par élément** en mmol/L : N, P, K, Ca, Mg, Fe, Mn, Mo, B, Cu, Zn, Co.
- **Cases vides à la source mises à 0** = élément absent de la recette (N pour IRR2 / Modified
  Hoagland, Ca/Mg/Co pour NPK). Corriger si des valeurs réelles existent.

## Variables du modèle

| Variable | Source | Remarque |
|---|---|---|
| N, P, K, Ca, Mg, Fe, Mn, Mo, B, Cu, Zn, Co | `medium_composition.csv` | Composition réelle |
| `light_lux` | Bloc du bac (1=1141, 2=834, 3=727) | Varie selon la position |
| `pH_mean` | Moyenne du pH par bac | Donnée observée |
| `M0` | Biomasse initiale réelle (v3) | Variable d'un bac à l'autre |
| `temp_mean`, `humidity_mean` | Capteur de salle (agrégé) | **Constantes** (variance nulle) → importance ~0, incluses à la demande |

**Non utilisées :** TDS, salinité (colinéaires à la CE), photopériode (constante).

## Conception expérimentale

24 bacs (réduits à 20), plan déséquilibré : NPK (contrôle), IRR2, Yoshida, Modified Hoagland.
Identifiant unique de bac = `Medium + Bloc + Bac`.
