# 🌍 Modèle de Régression Multilinéaire — Prédiction de l'Espérance de Vie

![Python](https://img.shields.io/badge/Python-3.10+-blue?logo=python)
![Statsmodels](https://img.shields.io/badge/Statsmodels-OLS-green)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-1.x-orange?logo=scikit-learn)
![Status](https://img.shields.io/badge/Status-Complété-brightgreen)

---

## 📌 Présentation

Ce projet implémente un **modèle de régression linéaire multiple (OLS)** pour prédire l'espérance de vie à partir de variables socio-économiques et sanitaires. La démarche est complète : exploration des données, analyse de corrélation, modélisation, comparaison de deux modèles et validation rigoureuse des hypothèses statistiques.

**Deux modèles sont construits et comparés :**
- **Modèle A** — 17 prédicteurs (toutes les variables numériques disponibles)
- **Modèle B** — 5 prédicteurs sélectionnés après analyse VIF (multicolinéarité maîtrisée)

---


## 📊 Dataset

| Caractéristique | Détail |
|---|---|
| **Source** | Life Expectancy Data — OMS / Banque Mondiale (version enrichie) |
| **Dimensions** | 2 864 observations × 20 colonnes |
| **Couverture** | Mondiale — 9 régions, années 2000–2015 |
| **Variable cible** | `Life_expectancy` (espérance de vie en années) |
| **Valeurs manquantes** | Aucune — dataset pré-traité |

### Variables principales

| Variable | Description |
|---|---|
| `Adult_mortality` | Taux de mortalité adulte (pour 1 000 hab.) |
| `Infant_deaths` | Mortalité infantile |
| `Under_five_deaths` | Mortalité enfants de moins de 5 ans |
| `Alcohol_consumption` | Consommation d'alcool moyenne |
| `Incidents_HIV` | Incidence du VIH |
| `GDP_per_capita` | PIB par habitant |
| `Population_mln` | Population en millions |
| `Schooling` | Années moyennes de scolarisation |
| `BMI` | Indice de masse corporelle moyen |
| `Polio`, `Diphtheria`, `Hepatitis_B` | Taux de vaccination |

---

## ⚙️ Pipeline d'analyse

### 1. Exploration et nettoyage
- Audit qualité : dimensions, types, valeurs manquantes (résultat : **0 valeur manquante**)
- Statistiques descriptives globales
- Distribution des variables (histogrammes + boxplots)

### 2. Analyse de corrélation
- Matrice de corrélation complète (heatmap)
- Nuages de points entre chaque prédicteur et `Life_expectancy`
- Variables avec corrélation **|r| > 0,6** avec la cible : `Infant_deaths`, `Under_five_deaths`, `Adult_mortality`, `Polio`, `Diphtheria`, `Schooling`
- Paires fortement corrélées entre elles (|r| > 0,8) : `Infant_deaths` / `Under_five_deaths`, `Polio` / `Diphtheria`, `Thinness_ten_nineteen_years` / `Thinness_five_nine_years`

### 3. Division Train / Test
- Split **80% / 20%** avec `random_state=42` pour assurer la reproductibilité
- Toute la sélection de variables est effectuée **uniquement sur le train** (pas de data leakage)

### 4. Modélisation OLS

**Modèle A — Toutes les variables (17 prédicteurs)**
Modèle de référence entraîné sur l'ensemble complet des variables numériques.

**Modèle B — Sélection par VIF (5 prédicteurs)**
Après analyse du Variance Inflation Factor (VIF > 46 sur `Infant_deaths`, VIF > 473 sur `Economy_status_Developing`), une sélection rigoureuse réduit le modèle à 5 variables avec des VIF tous inférieurs à 3 :

| Variable | VIF |
|---|---|
| `Adult_mortality` | 2.61 |
| `Alcohol_consumption` | 2.73 |
| `Incidents_HIV` | 1.79 |
| `GDP_per_capita` | 1.92 |
| `Population_mln` | 1.06 |

### 5. Évaluation et comparaison des modèles

| Métrique | Modèle A (17 vars) | Modèle B (5 vars) |
|---|---|---|
| **R² Train** | 0.98 | 0.94 |
| **R² Test** | 0.98 | 0.93 |
| **MSE Train** | 1.84 | 5.34 |
| **MSE Test** | 1.85 | 5.80 |
| **RMSE Train** | 1.36 an | 2.31 ans |
| **RMSE Test** | 1.36 an | 2.41 ans |

### 6. Validation des hypothèses (Modèle B)

| Hypothèse | Test | Résultat |
|---|---|---|
| Homoscédasticité | Breusch-Pagan | ⚠️ Rejetée — stat=252, p≈0 |
| Normalité des résidus | Shapiro-Wilk (stat=0.97, p≈0) | ⚠️ Rejetée |
| Normalité des résidus | Kolmogorov-Smirnov (stat=0.17, p≈0) | ⚠️ Rejetée |
| Normalité des résidus | Anderson-Darling (stat=12.84) | ⚠️ Rejetée |
| Autocorrélation | Durbin-Watson | ✅ 1.95 — absence d'autocorrélation |
| Multicolinéarité | VIF | ✅ Tous < 3 — excellente indépendance |

---

## ⚠️ Limites identifiées

**Hétéroscédasticité** — La variance des résidus n'est pas constante. Les prédictions sont moins précises aux valeurs extrêmes. Piste d'amélioration : transformation logarithmique de la cible ou erreurs robustes (HC3).

**Non-normalité des résidus** — Tous les tests formels rejettent l'hypothèse de normalité. Avec n > 2 800 observations, le théorème central limite garantit toutefois une bonne approximation asymptotique des estimateurs.

**Compromis performance / parcimonie** — Le Modèle B perd ~4 points de R² par rapport au Modèle A (0.93 vs 0.98), en échange d'une multicolinéarité quasi-nulle et d'une interprétabilité nettement supérieure (5 variables vs 17).

---

## 🛠️ Stack technique

| Librairie | Usage |
|---|---|
| `pandas`, `numpy` | Manipulation et traitement des données |
| `matplotlib`, `seaborn` | Visualisation (distributions, heatmaps, nuages de points) |
| `statsmodels` | Modélisation OLS, VIF, Breusch-Pagan, Durbin-Watson |
| `scikit-learn` | Split train/test, R², MSE, RMSE |
| `scipy.stats` | Shapiro-Wilk, Kolmogorov-Smirnov, Anderson-Darling, Q-Q plot |

---
| **Source du dataset** | [Life Expectancy Data — OMS / Banque Mondiale (version enrichie)](https://www.kaggle.com/datasets/lashagoch/life-expectancy-who-updated/) |


---

## 📄 Licence

Projet à des fins éducatives. Dataset issu de sources publiques (OMS / Banque Mondiale).
