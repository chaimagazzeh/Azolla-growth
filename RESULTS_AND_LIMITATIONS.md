# Approche 1 — Résultats et limites

Interprétation du notebook `notebooks/approach1_biomass_model.ipynb`.
Cible = **TCR** (taux de croissance relatif, Hunt 1982). Données : `dataset azolla_v3.xlsx`
(feuille `Feuil2`), **n=20** (plan 6/6/5 + NPK=3), composition élémentaire réelle (mode `composition`).
Chaque section renvoie à la figure / au fichier correspondant dans `outputs/`.

---

## Conditions de culture — figure `00_conditions_salle.png`

Capteur de salle agrégé par jour : **température ≈ 23 °C**, **humidité ≈ 64 %**, stables sur la
période (hors pic de mise en place du 11/05 à ~26 °C). Conclusion : environnement maîtrisé, sans
dérive notable. Ces valeurs **décrivent le contexte** mais ne servent pas de prédicteurs (capteur
unique → identiques pour tous les bacs ; voir importance nulle plus bas).

---

## Distribution du TCR — figure `01_eda.png`

- **Boxplot TCR par milieu :** les médianes se chevauchent largement entre milieux et la dispersion
  **intra-milieu** est importante (boîtes hautes). C'est le signal clé qui explique pourquoi la
  prédiction au niveau du bac reste difficile : la variabilité biologique entre répétitions d'un
  même milieu est du même ordre que les écarts entre milieux.
- **Nuage TCR vs pH moyen :** tendance faible mais visible — le pH apparaît comme un co-déterminant,
  ce que confirme l'importance des variables (plus bas).

---

## Performances des modèles — figure `02_benchmark_loocv.png`, fichier `model_comparison.csv`

LOOCV, n=20 :

| Modèle               | RMSE     | MAE      |
|----------------------|----------|----------|
| **Régression linéaire** | 0.00590 | 0.00462  |
| Random Forest        | 0.00593  | 0.00436  |
| Ridge                | 0.00615  | 0.00465  |
| XGBoost              | 0.00977  | 0.00871  |

Baseline (prédire le TCR moyen ≈ 0.0329 j⁻¹) : RMSE ≈ **0.00929**.

### Lecture (graphique de gauche : barres + ligne baseline)

- **Les trois modèles simples (linéaire, RF, Ridge) passent nettement à gauche de la baseline**
  (~0.0059–0.0062 vs 0.0093) : ils réduisent l'erreur d'environ **un tiers** → apport réel.
- La **régression linéaire est la meilleure** (RMSE la plus faible), devant Random Forest de peu.
  C'est le résultat attendu sous le **principe de parcimonie** : à n=20, un modèle simple et
  régularisé généralise mieux qu'un modèle complexe.
- **XGBoost est nettement le moins bon** (0.0098, ~baseline) : le boosting surajuste à cet effectif.

### Lecture (graphique de droite : observé vs prédit)

Les points du meilleur modèle se répartissent **autour de la diagonale**, sans nuage purement
horizontal → le modèle capte une part réelle de la variation du TCR (il ne se contente pas de
prédire la moyenne).

---

## Importance des variables — figure `03_shap_summary.png`, fichier `feature_importance.csv`

|SHAP| moyen (modèle retenu) — ordre décroissant :

| Variable | |SHAP| moyen | Interprétation |
|---|---|---|
| `M0` | 5.4e-3 | Biomasse initiale — en partie mécanique (terme du TCR), mais réelle et connue *a priori* |
| `pH_mean` | 4.3e-3 | **Second facteur** : le pH du milieu co-détermine la croissance |
| `light_lux` | 2.3e-3 | **Troisième facteur** : l'éclairement (bloc) influence le TCR |
| `P`, `Zn`, `Fe`, `N`, `Cu`, `Ca`, `K`… | 1.2e-3 → 7e-4 | Gradient nutritif — le phosphore en tête, cohérent avec la biologie d'*Azolla* |
| `temp_mean`, `humidity_mean` | 0 | Variance nulle (capteur de salle) → aucune contribution (attendu) |

**Insight majeur (vs versions précédentes) :** avec le plan équilibré 6/6/5, l'importance n'est
**plus monopolisée par M0**. Le `pH_mean` et la lumière émergent comme co-facteurs, et un **gradient
nutritif interprétable** apparaît (P, Fe, N…). Le modèle s'appuie donc sur des variables
**biologiquement sensées**, pas seulement sur l'algèbre du TCR.

### Le caveat M0 (toujours à mentionner)

Le TCR = (ln M21 − ln M0)/t contient M0. Son importance de tête est donc **en partie mécanique**.
Ce n'est **pas une fuite** (M0 est connue avant l'expérience), mais l'interprétation doit rester
prudente : c'est `pH_mean`, `light_lux` et le gradient nutritif qui portent le **sens biologique**.

---

## Réduction à n=20 — plan 6/6/5 (figure implicite : section 4 du notebook)

24 → 20 bacs pour cohérence avec le jeu biochimique. On vise le plan **le plus équilibré possible** :
NPK=3 conservé (contrôle), et les 17 bacs restants répartis en **6/6/5** (la partition entière la
plus équilibrée de 17 sur 3 milieux ; 6/6/6 impossible). Un plan équilibré est préférable au plan
5/5/7 (limite la domination d'un milieu dans l'erreur et l'ANOVA).

**Bacs retirés** (`RANDOM_STATE=42`, reproductible) :
1. `IRR2_Bloc1_B4` — pH **incomplet** (12/18 mesures).
2. `Modified Hoagland_Bloc2_B1`, `Modified Hoagland_Bloc2_B3`, `Yoshida_Bloc3_B2` — **tirés au
   hasard** (graine fixée) pour atteindre les tailles cibles, sans toucher au contrôle.

Effectif final : **NPK=3, IRR2=6, Yoshida=6, Modified Hoagland=5**.

---

## Variables incluses mais non discriminantes

`temp_mean`, `humidity_mean` : incluses à la demande (cohérence avec la liste d'entrées du mémoire),
mais **importance nulle** car identiques pour tous les bacs (capteur de salle). Résultat **attendu**,
pas une anomalie. `TDS`/`salinité` non utilisées (colinéaires à la CE). `Co` ≈ 0 (quasi absent des
recettes).

---

## Note sur la composition (`data/medium_composition.csv`)

Générée depuis la 3ᵉ feuille de la v3. Cases vides à la source mises à **0** (élément absent) : N pour
IRR2 et Modified Hoagland (milieu sans azote externe par conception), Ca/Mg/Co pour NPK. Corriger
dans le CSV si des valeurs réelles existent — le notebook les reprendra automatiquement.

---

## Approche 2 (déplétion de la CE) — non implémentée

L'estimation de k via CE(t) = CE₀·e^(−kt) exige **plusieurs mesures de CE par bac à des jours
différents**. Les données n'ont qu'**une valeur de CE par bac** → k indéfini. Conservée comme
**méthodologie proposée** (perspectives) ; aucune valeur n'est inventée.

---

## Synthèse pour la rédaction

> Le TCR d'*Azolla pinnata* est prédit avec une erreur LOOCV (~0.0059 j⁻¹) inférieure d'un tiers à
> la baseline (0.0093), par une **régression linéaire** — confirmant qu'un modèle simple suffit à
> cette échelle. Au-delà de la biomasse initiale (facteur en partie mécanique), le **pH** et
> l'**éclairement** ressortent comme co-déterminants, suivis d'un gradient nutritif cohérent
> (phosphore en tête). Les conditions ambiantes (température, humidité), homogènes, ne discriminent
> pas les bacs. La variabilité intra-milieu résiduelle (figure 01) borne la précision atteignable.

## Fichiers de sortie (`outputs/`)

| Fichier                    | Contenu                                            |
|----------------------------|----------------------------------------------------|
| `00_conditions_salle.png`  | Température/humidité journalières (capteur salle)  |
| `01_eda.png`               | Boxplot TCR/milieu + TCR vs pH                      |
| `02_benchmark_loocv.png`   | Comparaison RMSE des modèles + observé vs prédit   |
| `03_shap_summary.png`      | Importance des variables (SHAP) du meilleur modèle |
| `model_comparison.csv`     | Métriques LOOCV (RMSE, MAE)                         |
| `feature_importance.csv`   | Importance des variables                           |