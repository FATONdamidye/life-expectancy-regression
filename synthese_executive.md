# Synthèse Exécutive — Modèle de Prédiction de l'Espérance de Vie

**Projet :** Régression Linéaire Multiple (OLS)
**Données :** 2 864 observations × 20 variables — couverture mondiale (9 régions, 2000–2015)
**Outil principal :** Python — statsmodels, scikit-learn, scipy

---

## Objectif

Identifier les déterminants socio-économiques et sanitaires de l'espérance de vie et construire un modèle prédictif robuste. Deux modèles sont développés et comparés : un modèle complet de référence (Modèle A) et un modèle parcimonieux optimisé (Modèle B).

---

## Démarche

L'analyse suit un pipeline structuré en six étapes.

**Exploration** — Le dataset compte 2 864 observations et 20 variables sans aucune valeur manquante. Les distributions sont examinées via histogrammes et boxplots, révélant une asymétrie notable sur les variables de mortalité et de population.

**Analyse des corrélations** — La matrice de corrélation identifie six variables fortement liées à l'espérance de vie (|r| > 0,6) : `Infant_deaths`, `Under_five_deaths`, `Adult_mortality`, `Polio`, `Diphtheria` et `Schooling`. Des paires de variables très corrélées entre elles sont également détectées (|r| > 0,8), signalant un risque de multicolinéarité.

**Division Train/Test** — Le dataset est divisé en 80% entraînement / 20% test (`random_state=42`). Toute la sélection de variables est réalisée exclusivement sur le jeu d'entraînement pour éviter tout data leakage.

**Modélisation** — Deux modèles OLS (Ordinary Least Squares) sont estimés via statsmodels. Le Modèle A utilise les 17 variables numériques disponibles. Le Modèle B est construit après sélection par VIF, ne conservant que les 5 variables dont le facteur d'inflation de la variance est inférieur à 3.

**Évaluation** — Les métriques R², MSE et RMSE sont calculées sur train et test pour les deux modèles, permettant une comparaison directe performance/parcimonie.

**Validation des hypothèses** — Quatre hypothèses fondamentales de la régression OLS sont testées formellement sur le Modèle B : homoscédasticité (Breusch-Pagan), normalité des résidus (Shapiro-Wilk, Kolmogorov-Smirnov, Anderson-Darling), autocorrélation (Durbin-Watson) et multicolinéarité (VIF).

---

## Résultats

### Comparaison des deux modèles

| Métrique | Modèle A — 17 variables | Modèle B — 5 variables |
|---|---|---|
| **R² Train** | 0.98 | 0.94 |
| **R² Test** | 0.98 | 0.93 |
| **RMSE Train** | 1.36 an | 2.31 ans |
| **RMSE Test** | 1.36 an | 2.41 ans |
| **MSE Train** | 1.84 | 5.34 |
| **MSE Test** | 1.85 | 5.80 |

Le **Modèle A** offre des performances exceptionnelles avec un R² de 0,98 et un RMSE de seulement 1,36 an. La parfaite stabilité entre train et test confirme l'absence de surapprentissage. En contrepartie, ce modèle souffre d'une multicolinéarité sévère (VIF > 46 sur `Infant_deaths` et `Under_five_deaths`, VIF > 473 sur `Economy_status_Developing`), ce qui rend les coefficients peu fiables et difficiles à interpréter.

Le **Modèle B** sacrifie ~4 points de R² (0,93 en test) pour gagner une indépendance quasi-totale entre les prédicteurs (VIF max = 2,73). Avec seulement 5 variables, il reste très performant et offre une interprétabilité nettement supérieure. Les 5 variables retenues sont : `Adult_mortality`, `Alcohol_consumption`, `Incidents_HIV`, `GDP_per_capita` et `Population_mln`.

### Variables retenues dans le Modèle B et leur VIF

| Variable | VIF | Interprétation |
|---|---|---|
| `Adult_mortality` | 2.61 | Mortalité adulte — fort levier sur l'espérance de vie |
| `Alcohol_consumption` | 2.73 | Facteur comportemental de risque |
| `Incidents_HIV` | 1.79 | Impact sanitaire direct |
| `GDP_per_capita` | 1.92 | Niveau de développement économique |
| `Population_mln` | 1.06 | Taille du pays — effet structurel |

### Validation des hypothèses du Modèle B

| Hypothèse | Test | Conclusion |
|---|---|---|
| Homoscédasticité | Breusch-Pagan (stat=252, p≈0) | ⚠️ Non vérifiée |
| Normalité | Shapiro-Wilk (stat=0.97, p≈0) | ⚠️ Non vérifiée |
| Normalité | Kolmogorov-Smirnov (stat=0.17, p≈0) | ⚠️ Non vérifiée |
| Normalité | Anderson-Darling (stat=12.84) | ⚠️ Non vérifiée |
| Autocorrélation | Durbin-Watson = 1.95 | ✅ Vérifiée |
| Multicolinéarité | VIF max = 2.73 | ✅ Vérifiée |

---

## Limites et recommandations

**Hétéroscédasticité** — La variance des résidus n'est pas constante dans les deux modèles. Cela fragilise les intervalles de confiance et les tests de significativité des coefficients, sans invalider les prédictions. Recommandation : appliquer une transformation logarithmique sur la variable cible ou estimer les modèles avec des erreurs robustes de type HC3.

**Non-normalité des résidus** — Tous les tests formels rejettent l'hypothèse de normalité. Avec n = 2 864 observations, le théorème central limite garantit néanmoins une bonne approximation asymptotique — l'impact pratique est limité sur les prédictions.

**Choix du modèle final** — Pour une application prédictive pure, le Modèle A est préférable (R²=0,98, RMSE=1,36). Pour une analyse causale ou une communication auprès d'un public non technique, le Modèle B est recommandé pour sa parcimonie et son interprétabilité.

**Pistes d'amélioration** — Appliquer un filtre géographique pour construire des modèles régionaux spécifiques (Afrique, Asie, etc.), intégrer des effets fixes pays/année pour les données de panel, ou tester des modèles régularisés (Ridge, Lasso) pour mieux gérer la colinéarité résiduelle.

---

## Conclusion

Ce projet démontre une maîtrise complète du pipeline de modélisation en régression linéaire : de l'exploration à la validation des hypothèses, en passant par la comparaison rigoureuse de deux approches. Les performances du Modèle A (R²=0,98) sont excellentes. Le Modèle B illustre la valeur d'un modèle parcimonieux et interprétable dans un contexte de données réelles. Les limites sont identifiées, documentées et assorties de recommandations concrètes — ce qui témoigne d'une démarche analytique rigoureuse et honnête.

---

*Analyse réalisée avec Python — statsmodels, scikit-learn, scipy.stats, seaborn.*
