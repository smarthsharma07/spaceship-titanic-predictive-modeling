# 🚀 Spaceship Titanic: An Experiment-Driven Predictive Modeling Project

## Overview

This project explores the **Spaceship Titanic** Kaggle competition, where the objective is to predict whether a passenger was transported to an alternate dimension during an interstellar voyage.

Rather than focusing solely on model complexity, this project investigates the impact of:

- Feature Engineering
- Data Preprocessing Strategies
- Model Selection
- Hyperparameter Optimization
- Systematic Experiment Tracking

The final solution achieved a **Public Leaderboard Score of 0.80780**, placing approximately **497th out of 2500+ participants (Top ~20%)**, using a single CatBoost model.

---

## Problem Statement

The task is to predict the binary target:

```text
Transported
```

using passenger information such as:

- HomePlanet
- Destination
- Cabin Location
- Spending Habits
- CryoSleep Status
- VIP Status
- Age
- Travel Groups

---

## Dataset

The dataset contains passenger records from the Spaceship Titanic voyage.

Key attributes include:

| Category | Features |
|-----------|-----------|
| Demographics | Age, HomePlanet |
| Travel Information | Destination, Cabin |
| Passenger Status | CryoSleep, VIP |
| Spending Behavior | RoomService, FoodCourt, ShoppingMall, Spa, VRDeck |
| Group Information | PassengerId |

Competition:

**Spaceship Titanic (Kaggle)**

---

## Project Goals

The primary objectives were:

- Build a strong tabular machine learning pipeline.
- Explore feature engineering techniques.
- Evaluate CatBoost against alternative approaches.
- Understand the relationship between cross-validation and leaderboard performance.
- Develop a reproducible experimentation workflow.

---

## Technologies Used

- Python
- Pandas
- NumPy
- Scikit-Learn
- CatBoost
- LightGBM
- Optuna
- Matplotlib
- Seaborn

---

## Feature Engineering

### Cabin Features

The original cabin field was decomposed into:

```python
Deck
CabinNum
Side
```

Additional engineered features:

```python
DeckSide
CabinNumBucket
```

These features capture passenger location information within the ship.

---

### Group Features

Passenger groups were extracted from PassengerId.

Created:

```python
GroupID
GroupSize
IsAlone
```

These features help model relationships among passengers traveling together.

---

### Spending Features

Combined spending categories into:

```python
TotalSpend
ZeroSpend
LogTotalSpend
```

These became some of the strongest predictors in the final model.

---

### Missing Value Indicators

Created binary indicators for missing values:

```python
AgeMissing
CryoSleepMissing
VIPMissing
HomePlanetMissing
DestinationMissing
CabinMissing
```

This preserved information about missingness before imputation.

---

## Modeling Approach

### Primary Model

```python
CatBoostClassifier
```

Reasons for selecting CatBoost:

- Native categorical feature support
- Strong tabular-data performance
- Minimal preprocessing requirements
- Robust handling of feature interactions

---

## Experimental Timeline

| Experiment | Public Score |
|------------|-------------:|
| Baseline | 0.80383 |
| + DeckSide + CabinNumBucket | 0.80360 |
| - LuxurySpend - BasicSpend | 0.80453 |
| + HomePlanet_Deck | 0.80313 |
| + SpendPerPerson | 0.80430 |
| Feature Engineering Before Imputation | 0.80593 |
| Remove VIP + IsAlone | **0.80780** |

---

## Major Experiments

### Pipeline Order Change

#### Hypothesis

Creating features before imputation may preserve more useful information than imputing first.

#### Original Pipeline

```text
Imputation
↓
Feature Engineering
↓
Model
```

#### Revised Pipeline

```text
Feature Engineering
↓
Imputation
↓
Model
```

#### Result

```text
0.80593
```

This became one of the most impactful improvements during the competition.

---

### Route Feature

Created:

```python
Route = HomePlanet + "_" + Destination
```

#### Hypothesis

Passenger origin and destination combinations may contain predictive signal.

#### Result

```text
0.80780 → 0.80336
```

#### Conclusion

The feature introduced redundancy and likely duplicated interactions already learned by CatBoost.

---

### Cabin Bucket Granularity

Changed:

```python
CabinNumBucket = CabinNum // 100
```

to:

```python
CabinNumBucket = CabinNum // 50
```

#### Hypothesis

Finer location groupings may improve prediction quality.

#### Result

```text
0.80780 → 0.80266
```

#### Conclusion

Smaller buckets introduced noise and reduced generalization.

---

### Group Feature Modifications

#### Hypothesis

Low-importance group features might be unnecessary.

#### Result

```text
0.80780 → 0.80547
```

#### Conclusion

Even low-importance features can contribute useful signal.

---

## Hyperparameter Optimization

Several tuning approaches were explored.

### RandomizedSearchCV

Multiple CatBoost parameters were evaluated.

Result:

No meaningful leaderboard improvement.

---

### Optuna

Performed 50 optimization trials.

Best Cross-Validation Score:

```text
0.81564
```

Leaderboard Result:

```text
0.80366
```

### Key Observation

A higher cross-validation score did not necessarily translate to a better leaderboard score.

---

### GridSearchCV

The most extensive tuning experiment.

Leaderboard Score:

```text
0.80149
```

This became one of the lowest-performing submissions despite requiring the most computational effort.

---

## Model Comparison

### CatBoost

Best Public Score:

```text
0.80780
```

### LightGBM

Average Performance:

```text
0.803 - 0.804
```

The results suggested that feature quality had a larger impact than model choice.

---

## Final Feature Importance

| Feature | Importance |
|----------|-----------:|
| Spa | 13.44 |
| VRDeck | 9.79 |
| HomePlanet | 9.69 |
| FoodCourt | 9.40 |
| DeckSide | 7.69 |
| CabinNumBucket | 7.48 |
| TotalSpend | 6.27 |
| RoomService | 6.21 |
| ShoppingMall | 5.24 |
| Deck | 4.97 |
| Age | 4.67 |
| CryoSleep | 4.47 |

Lowest contributing features:

| Feature | Importance |
|----------|-----------:|
| GroupSize | 0.276 |
| GroupID | 0.274 |

---

## Biggest Breakthrough

Feature importance analysis revealed:

| Feature | Importance |
|----------|-----------:|
| VIP | ~0.04 |
| IsAlone | ~0.04 |

#### Hypothesis

These features may be contributing more noise than signal.

#### Action

Removed:

```python
VIP
IsAlone
```

#### Result

```text
0.80593 → 0.80780
```

This became the best-performing model of the competition.

---

## Key Findings

### Feature Engineering > Hyperparameter Tuning

Most leaderboard improvements came from improving features rather than tuning model parameters.

---

### More Features ≠ Better Model

Several engineered features increased complexity without improving performance.

Examples:

- Route Features
- HomePlanet_Deck
- Aggressive Cabin Bucketing

---

### Cross Validation ≠ Leaderboard Performance

The highest cross-validation score obtained during optimization did not produce the best public leaderboard result.

---

### Feature Importance ≠ Leaderboard Importance

Some low-importance features improved performance, while some highly important features reduced leaderboard score.

---

## Final Results

| Metric | Value |
|----------|----------|
| Model | CatBoost |
| Public Leaderboard Score | **0.80780** |
| Approximate Rank | **497 / 2500+** |
| Percentile | **Top ~20%** |

Achieved using:

- Single CatBoost Model
- Feature Engineering
- Simple Preprocessing
- No Ensembling
- No Stacking
- No Neural Networks

---

## Project Structure

```text
spaceship-titanic-predictive-modeling/
│
├── notebook.ipynb
├── README.md
├── requirements.txt
├── spaceship-titanic-transport-dataset/
│   ├── train.csv
│   ├── test.csv
│   └── sample_submission.csv
│
└── submission.csv
```

---


## Acknowledgements

- Kaggle for hosting the Spaceship Titanic competition.
- The Kaggle community for public notebooks and discussions that inspired several experimental directions.

---

## Final Thoughts

This project demonstrated that strong performance on tabular machine learning tasks often comes from careful feature engineering, disciplined experimentation, and understanding the data rather than relying solely on increasingly complex models.

**Final Public Leaderboard Score:** **0.80780**

**Best Rank Achieved:** **~497 / 2500+ Participants**

A project showcasing how thoughtful experimentation and feature engineering can outperform extensive hyperparameter optimization on real-world tabular data.

- Smarth Sharma 
