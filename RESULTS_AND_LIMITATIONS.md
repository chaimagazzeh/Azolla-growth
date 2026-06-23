# Approche 1 — Résultats et limites

Document d'interprétation du notebook `notebooks/approach1_biomass_model.ipynb`.
Cible = **TCR** (taux de croissance relatif, Hunt 1982). Chiffres obtenus en mode **one-hot**
(CSV de composition non encore rempli) ; à recalculer une fois `medium_composition.csv` complété.

## Résultats de modélisation (LOOCV, n=24)

| Modèle               | RMSE     | MAE      |
|----------------------|----------|----------|
| Ridge                | 0.01094  | 0.00947  |
| Régression linéaire  | 0.01098  | 0.00840  |
| XGBoost              | 0.01113  | 0.00991  |
| Random Forest        | 0.01178  | 0.01017  |

Baseline (prédire le TCR moyen ≈ 0.0326 j⁻¹) : RMSE ≈ **0.01067**.

### Lecture

- Les modèles se tiennent tous **autour de la baseline** (≈ 0.0107). Ridge fait marginalement
  mieux, la régression linéaire a le plus faible MAE ; les modèles complexes (RF, XGBoost) ne
  font pas mieux.
- Conforme au **principe de parcimonie** : à très petit n, augmenter la complexité algorithmique
  n'apporte aucun gain. Les modèles linéaires régularisés sont les plus appropriés.
- **Constat honnête :** le TCR d'un bac individuel n'est pas finement prédictible à partir des
  seules variables disponibles ; la variabilité biologique entre répétitions domine.

## Pourquoi les performances restent proches de la baseline — explication factuelle

1. **Seulement 4 profils de conditions distincts.** Les 24 bacs se répartissent sur 4 milieux. En
   one-hot, les bacs d'un même milieu ont des features quasi identiques → le modèle apprend ~4
   moyennes de groupe, pas une relation fine.
2. **Forte variabilité intra-milieu.** Pour un même milieu, le TCR varie sensiblement d'une
   répétition à l'autre. Cette dispersion n'est explicable par aucune variable mesurée.
3. **M0 constante (36 g).** Incluse comme variable conformément au mémoire, mais sans pouvoir
   discriminant (variance nulle, neutralisée par la standardisation). Le TCR est donc, à un
   facteur près, une transformation de la seule biomasse finale.

## Variables d'entrée : justifications

| Variable      | Statut    | Justification                                                                 |
|---------------|-----------|------------------------------------------------------------------------------|
| Composition / one-hot | Incluse | Encode le milieu (vraies concentrations dès que le CSV est rempli).   |
| `light_lux`   | Incluse   | Varie selon la table (T1=1141, T2=834, T3=727) → information réelle.          |
| `pH_mean`     | Incluse   | pH **moyen observé** par bac. On utilise la donnée effectivement mesurée ; faute de pH cible distinct enregistré par bac, le pH moyen est le meilleur proxy disponible. |
| `M0`          | Incluse   | Biomasse initiale, listée comme entrée dans le mémoire. Constante (36 g) → sans effet discriminant mais sans nuisance. |

## Variables exclues — et pourquoi

| Variable          | Raison de l'exclusion                                                              |
|-------------------|-----------------------------------------------------------------------------------|
| Température        | Non disponible par bac (condition de salle) → pas de variance exploitable entre bacs. Ne peut pas servir de variable discriminante. |
| Humidité          | Idem : non mesurée par bac / constante au niveau de la salle → variance nulle.     |
| Photopériode      | Constante sur toute l'expérience et tous les bacs.                                 |
| TDS / Salinité    | Dérivés directs de la CE (colinéarité ≈ 1) → redondants.                           |

> Ces variables figurent dans la description du dispositif mais ne peuvent pas servir de
> prédicteurs faute de variation entre bacs (ou par redondance). À déclarer dans les résultats.

## Périmètre d'échantillon : 16 vs 24 bacs

- **Analyses biochimiques : 16 bacs** (seuls bacs dosés biochimiquement).
- **Modélisation : 24 bacs** (tous ceux disposant d'une biomasse finale). À si petite taille,
  inclure toutes les observations valides maximise l'échantillon et renforce la LOOCV ; aucune
  donnée exploitable n'est écartée. À défendre explicitement comme choix méthodologique.

## Approche 2 (déplétion de la CE) — discutée théoriquement, non implémentée

L'estimation de la constante k via CE(t) = CE₀·e^(−kt) exige **plusieurs mesures de CE à des
jours différents par bac**. Les données ne contiennent qu'**une valeur de CE par bac** (pas de
série temporelle exploitable) → impossible d'ajuster k. L'approche est donc présentée comme
**cadre méthodologique proposé** ; sa validation empirique nécessiterait un échantillonnage CE
plus dense (perspectives / travaux futurs). Aucune valeur de k n'est calculée ni inventée.

## Amélioration possible (sans surinterpréter)

- **Renseigner `medium_composition.csv`** : remplace le one-hot par les concentrations réelles.
  N'augmentera pas la performance (toujours 4 profils) mais rendra l'importance des variables
  (SHAP) **biologiquement interprétable** (P, N, K… au lieu d'indicatrices).
- **Cadrage mémoire** : la biomasse/TCR est déterminée par le milieu au niveau *du groupe*
  (ANOVA) mais non prédictible au niveau *du bac individuel* à partir de ces seules entrées.

## Fichiers de sortie (`outputs/`)

| Fichier                    | Contenu                                            |
|----------------------------|----------------------------------------------------|
| `01_eda.png`               | Boxplot TCR/milieu + TCR vs pH                      |
| `02_benchmark_loocv.png`   | Comparaison RMSE des modèles + observé vs prédit   |
| `03_shap_summary.png`      | Importance des variables (SHAP) du meilleur modèle |
| `model_comparison.csv`     | Tableau des métriques LOOCV (RMSE, MAE)            |
| `feature_importance.csv`   | Importance des variables                           |
