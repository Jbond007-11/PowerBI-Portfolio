# NHL Goals Prediction Model - Detailed Explanation
## Assignment 5: Machine Learning for Player Performance Forecasting

---

## Table of Contents
1. [Introduction & Overview](#introduction--overview)
2. [Cell-by-Cell Breakdown](#cell-by-cell-breakdown)
3. [Key Concepts Explained](#key-concepts-explained)
4. [Summary of Results](#summary-of-results)

---

## Introduction & Overview

### What is this notebook doing?
This notebook uses **machine learning** to predict how many goals NHL hockey players will score in their **next season** based on their **past performance**. Think of it like a weather forecast, but instead of predicting rain, we're predicting player performance.

### Why is this useful?
Hockey teams can use these predictions to:
- Decide which players to trade for or sign
- Negotiate fair contracts based on expected performance
- Plan team strategies and lineups
- Identify players who might improve or decline

### The Big Picture Process
1. **Collect Data**: Get historical stats for NHL forwards (2015-2024)
2. **Clean Data**: Remove errors and missing values
3. **Create Features**: Calculate useful metrics from raw stats
4. **Train Model**: Teach the computer to recognize patterns
5. **Test Model**: See how well it predicts new data
6. **Evaluate**: Measure accuracy and understand strengths/weaknesses

---

## Cell-by-Cell Breakdown

---

### **Cell 0: Introduction (Markdown)**

#### What it says:
This is the title page explaining the project goals and student information.

#### Key Points:
- **Goal**: Predict future NHL player goal production
- **Method**: Linear regression (a type of machine learning)
- **Data**: NHL player stats from 2015-2024
- **Research Question**: "Can we accurately predict a player's future goal production using their historical performance metrics?"

#### Why this matters:
Setting clear objectives helps guide the entire analysis. We know exactly what we're trying to accomplish.

---

### **Cell 1: Step 1 Title (Markdown)**

Just a section header - no code to explain here.

---

### **Cell 2: Import Libraries**

#### The Code:
```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import joblib
```

#### What this does:
Loads all the tools (libraries) we need for the project. Think of it like gathering all your ingredients before cooking.

#### Library Explanations:
- **pandas** (`pd`): Works with data tables (like Excel spreadsheets)
- **numpy** (`np`): Does mathematical calculations on numbers
- **matplotlib.pyplot** (`plt`): Creates charts and graphs
- **seaborn** (`sns`): Makes prettier, more professional-looking charts
- **sklearn tools**: Machine learning toolkit
  - `train_test_split`: Splits data into practice and test sets
  - `LinearRegression`: The prediction model we'll use
  - `StandardScaler`: Makes all numbers similar in size (important for accuracy)
  - Metrics: Tools to measure how good our predictions are
- **joblib**: Saves our trained model for later use

#### The Output:
```
Libraries loaded successfully
```
This confirms all tools loaded without errors.

---

### **Cell 3: Step 2.1 Title (Markdown)**

Section header explaining we're about to load the dataset.

---

### **Cell 4: Load the Dataset**

#### The Code:
```python
df = pd.read_csv('combined_player_data.csv')
print(f'Dataset loaded: {df.shape[0]:,} rows & {df.shape[1]:,} columns')
df.head()
```

#### What this does:
1. Reads the CSV file (a spreadsheet) into a variable called `df` (short for "dataframe")
2. Prints how many rows (player-seasons) and columns (statistics) we have
3. Shows the first 5 rows so we can see what the data looks like

#### Understanding the Output:
The output would show something like:
- **Rows**: Each row represents one player in one season
- **Columns**: Each column is a different statistic (goals, assists, shots, etc.)
- The table preview shows actual player names and their stats

#### Why this matters:
We need to understand the structure of our data before we can work with it. It's like looking at a map before starting a road trip.

---

### **Cell 5: Step 2.2 Title (Markdown)**

Explains we're about to filter the data to focus only on relevant players and situations.

---

### **Cell 6: Filter Data for Forwards Only**

#### The Code:
```python
forward_positions = ['C', 'L', 'R']
df_forwards = df[
    (df['situation'] == 'all') &
    (df['season'] >= 2015) &
    (df['position'].isin(forward_positions))
].copy()
```

#### What this does:
Filters the data to include only:
- **Forwards** (positions C=Center, L=Left Wing, R=Right Wing) - we exclude defensemen
- **"All situations"** (includes 5-on-5, power play, penalty kill combined)
- **Seasons 2015 onwards** (more recent, relevant data)

#### The Output:
```
Filtered data: 6,009 rows & 154 columns
Unique players: 1,378
Seasons covered: 2015 - 2024
```

#### What this means:
- **6,009 rows**: That's 6,009 "player-seasons" (e.g., Connor McDavid in 2023 is one row)
- **1,378 unique players**: Different individual players
- **154 columns**: Different statistics tracked for each player

#### Why filter?
- **Forwards score more goals** than defensemen, so they're more relevant for goal prediction
- **All situations** gives us complete scoring totals
- **Recent seasons** are more relevant to modern hockey

---

### **Cell 7: Step 2.3 Title (Markdown)**

Explains we're selecting only the most important features (statistics) for our model.

---

### **Cell 8: Select Relevant Features**

#### The Code:
```python
columns_to_select = [
    'season', 'name', 'team', 'position', 'games_played',
    'I_F_goals', 'I_F_primaryAssists', 'I_F_secondaryAssists',
    'I_F_xGoals', 'I_F_shotsOnGoal', 'I_F_highDangerShots',
    'icetime', 'I_F_points'
]
subset_df = df_forwards[columns_to_select].copy()
```

#### What this does:
Narrows down from 154 columns to just 13 most important ones.

#### Column Meanings:
- **Identifying info**: season, name, team, position
- **Playing time**: games_played, icetime (in seconds)
- **Scoring stats**:
  - `I_F_goals`: Actual goals scored
  - `I_F_primaryAssists`: First assist on a goal (passed to scorer)
  - `I_F_secondaryAssists`: Second assist
  - `I_F_points`: Total points (goals + assists)
- **Advanced metrics**:
  - `I_F_xGoals`: "Expected goals" - how many goals they *should* score based on shot quality
  - `I_F_shotsOnGoal`: Total shots on net
  - `I_F_highDangerShots`: Dangerous scoring chances

#### The Output:
```
Feature-selected data: 6,009 rows & 13 columns
```
Plus a preview table showing these columns.

#### Why select features?
Too many columns create "noise" and slow down the model. We keep only the most predictive statistics.

---

### **Cell 9: Step 2.4 Title (Markdown)**

We're about to check the data quality to ensure there are no errors or missing values.

---

### **Cell 10: Data Quality Checks**

#### The Code:
```python
print("Missing Values:")
print(subset_df.isna().sum())
print("Data Types:")
print(subset_df.dtypes)
print("Basic Statistics:")
subset_df.describe()
```

#### What this does:
Three quality checks:

1. **Missing Values Check**: Counts how many blank/missing values in each column
2. **Data Types Check**: Verifies each column has the right type (numbers, text, etc.)
3. **Basic Statistics**: Shows min, max, average, etc. for each number column

#### The Output Explained:

**Missing Values:**
```
season                  0
name                    0
...
I_F_points              0
```
✅ **Good news**: Zero missing values in all columns - our data is complete!

**Data Types:**
```
season                    int64    (whole numbers)
name                     object    (text)
I_F_goals               float64    (decimal numbers)
```
Each column has the appropriate type for its data.

**Basic Statistics Table:**
```
            season  games_played    I_F_goals  ...
mean   2019.552005     48.465302    10.214844  ...
std       2.868222     28.654373    10.301559  ...
min    2015.000000      1.000000     0.000000  ...
max    2024.000000     84.000000    69.000000  ...
```

#### What these statistics mean:
- **mean**: Average value (e.g., average player scored 10.2 goals)
- **std**: Standard deviation - how spread out the numbers are
- **min**: Lowest value (someone scored 0 goals)
- **max**: Highest value (someone scored 69 goals in a season!)
- **25%, 50%, 75%**: Quartiles showing distribution

#### Why this matters:
Understanding your data's shape helps you spot errors (like a player with -5 goals, which is impossible).

---

### **Cell 11: Step 3.1 Title (Markdown)**

We're now starting **Feature Engineering** - creating new, more useful statistics from the raw data.

---

### **Cell 12: Create Engineered Features**

#### The Code:
```python
subset_df['Goals_Per_Game'] = subset_df['I_F_goals'] / subset_df['games_played']
subset_df['Shooting_Pct'] = (subset_df['I_F_goals'] / subset_df['I_F_shotsOnGoal']) * 100
subset_df['Shot_Quality'] = subset_df['I_F_highDangerShots'] / subset_df['I_F_shotsOnGoal']
subset_df['Goals_Above_Expected'] = subset_df['I_F_goals'] - subset_df['I_F_xGoals']
subset_df['Ice_Time_Per_Game'] = subset_df['icetime'] / subset_df['games_played']
```

#### What this does:
Creates 5 new calculated columns (features) that give better insights than raw numbers.

#### Feature Explanations:

1. **Goals_Per_Game (GPG)**
   - Formula: Total goals ÷ Games played
   - Why: A player with 20 goals in 40 games (0.5 GPG) is better than 20 goals in 80 games (0.25 GPG)
   - Normalizes performance by playing time

2. **Shooting_Pct (Shooting Percentage)**
   - Formula: (Goals ÷ Shots on Goal) × 100
   - Why: Shows finishing skill - how often shots become goals
   - A sniper might have 15% shooting percentage vs. 8% for an average player

3. **Shot_Quality**
   - Formula: High danger shots ÷ Total shots
   - Why: Measures if a player gets good scoring chances
   - 0.20 means 20% of their shots are high-danger

4. **Goals_Above_Expected**
   - Formula: Actual goals - Expected goals
   - Why: Shows if a player exceeds or underperforms their opportunities
   - Positive = overperforming, Negative = underperforming

5. **Ice_Time_Per_Game**
   - Formula: Total ice time ÷ Games played
   - Why: More ice time often means more opportunities to score
   - Star players might average 20+ minutes per game

#### The Output:
```
Engineered features created successfully

Sample of new features:
                  name  season  I_F_goals  Goals_Per_Game  Shooting_Pct  Shot_Quality
1          Mikko Koivu    2015       17.0        0.207317     12.056738      0.156028
16      Filip Forsberg    2015       33.0        0.402439     13.360324      0.113360
```

#### Reading the sample:
- **Mikko Koivu**: 17 goals, 0.21 goals per game, 12.1% shooting percentage
- **Filip Forsberg**: 33 goals, 0.40 goals per game (almost double Koivu's rate!), 13.4% shooting

#### Why engineer features?
Machine learning models often work better with **ratios and comparisons** rather than raw totals. These engineered features capture important patterns.

---

### **Cell 13: Step 3.2 Title (Markdown)**

Now we'll clean up any problems that occurred during feature engineering.

---

### **Cell 14: Remove Missing Values**

#### The Code:
```python
print("Missing values after feature engineering:")
print(subset_df.isna().sum())

subset_df_clean = subset_df.dropna().copy()

print(f"\nClean dataset: {len(subset_df_clean):,} rows retained")
print(f"Rows removed: {len(subset_df) - len(subset_df_clean):,}")
```

#### What this does:
1. Checks for missing values (NaN = "Not a Number")
2. Removes any rows with missing values
3. Reports how many rows were kept vs. removed

#### The Output:
```
Missing values after feature engineering:
season                    0
name                      0
...
Shooting_Pct            166  ← Problem columns!
Shot_Quality            166  ← Problem columns!
...

Clean dataset: 5,843 rows retained
Rows removed: 166
```

#### Why do we have missing values now?
When we calculated `Shooting_Pct` and `Shot_Quality`, some players had **0 shots on goal** (they barely played). Dividing by zero creates an undefined value (NaN).

**Example**: A player played 2 games, never shot the puck → 0 goals ÷ 0 shots = NaN

#### Why remove these rows?
Players with 0 shots aren't useful for predicting goal-scorers. They're outliers who barely played.

#### Impact:
Lost 166 out of 6,009 rows = **2.8%** of data. This is acceptable since these were incomplete records.

---

### **Cell 15: Step 4.1 Title (Markdown)**

We're now preparing data specifically for machine learning by creating our **target variable** (what we want to predict).

---

### **Cell 16: Create Target Variable**

#### The Code:
```python
subset_df_clean = subset_df_clean.sort_values(['name', 'season']).reset_index(drop=True)

subset_df_clean['Next_Season_Goals'] = subset_df_clean.groupby('name')['I_F_goals'].shift(-1)

df_model = subset_df_clean.dropna(subset=['Next_Season_Goals']).copy()
```

#### What this does (step by step):

**Step 1: Sort the data**
```python
sort_values(['name', 'season'])
```
Organizes data by player name alphabetically, then by season chronologically.

**Before sorting:**
```
Connor McDavid, 2017
Sidney Crosby, 2015
Connor McDavid, 2015
```

**After sorting:**
```
Connor McDavid, 2015
Connor McDavid, 2017
Sidney Crosby, 2015
```

**Step 2: Create the target**
```python
subset_df_clean['Next_Season_Goals'] = subset_df_clean.groupby('name')['I_F_goals'].shift(-1)
```

This is **critical** - it creates a new column showing next season's goals for each player.

**Example:**
```
Player          Season  I_F_goals  Next_Season_Goals
Connor McDavid  2015    16         →  30  (his 2016 goals)
Connor McDavid  2016    30         →  41  (his 2017 goals)
Connor McDavid  2017    41         →  NaN (no 2018 data)
```

**Step 3: Remove incomplete rows**
```python
dropna(subset=['Next_Season_Goals'])
```
Removes the last season for each player (since we don't know their *next* season yet).

#### The Output:
```
Modeling dataset: 4,504 player-seasons
Players included: 1,031

            name  season  I_F_goals  Next_Season_Goals
0     A.J. Greer    2016        0.0                0.0
1     A.J. Greer    2017        0.0                1.0  ← He scored 1 goal in 2018
2     A.J. Greer    2018        1.0                1.0
3     A.J. Greer    2021        1.0                5.0  ← Big jump to 5 goals!
```

#### Understanding the sample:
- **Row 1**: A.J. Greer scored 0 goals in 2017, and we know he scored 0 in 2018 (next season)
- **Row 2**: He scored 0 in 2017, but 1 in 2018 (slight improvement)
- **Row 4**: He scored 1 in 2021, then jumped to 5 in 2022 (breakthrough season!)

#### Why this is important:
Our model will learn: "Given a player's current season stats, what will they score next year?"

This is called **supervised learning** - we teach the model using examples where we know the answer.

---

### **Cell 17: Step 4.2 Title (Markdown)**

Explains we're selecting which statistics (features) to use for predictions.

---

### **Cell 18: Select Features for Training**

#### The Code:
```python
feature_columns = [
    'games_played',
    'I_F_goals',
    'I_F_primaryAssists',
    'I_F_xGoals',
    'I_F_shotsOnGoal',
    'I_F_highDangerShots',
    'Goals_Per_Game',
    'Shooting_Pct',
    'Shot_Quality',
    'Goals_Above_Expected',
    'Ice_Time_Per_Game'
]

target = 'Next_Season_Goals'

X = df_model[feature_columns].copy()
y = df_model[target].copy()
```

#### What this does:

**1. Define features (predictors):**
These are the 11 statistics we'll use to make predictions. Think of them as "clues" the model uses.

**2. Define target:**
`Next_Season_Goals` is what we're trying to predict (the answer).

**3. Create X and y:**
- **X** (capital X): The feature matrix - all 11 columns for 4,504 players
- **y** (lowercase y): The target vector - just the goals we're predicting

#### The Output:
```
Feature matrix shape: (4504, 11)
Target vector shape: (4504,)

Features selected: 11
```

#### Understanding the shapes:
- **X = (4,504, 11)**: 4,504 rows (player-seasons) × 11 columns (features)
- **y = (4,504,)**: 4,504 values (one goal total per player-season)

#### Analogy:
Think of this like a multiple-choice test:
- **X** = The questions and available information
- **y** = The correct answers
- The model studies the patterns between questions and answers to predict new answers

#### Important note:
We're using **this season's stats** (X) to predict **next season's goals** (y). We never use next season's stats to predict next season's goals - that would be cheating (called "data leakage").

---

### **Cell 19: Step 4.3 Title (Markdown)**

Explains the crucial concept of splitting data into training and testing sets.

---

### **Cell 20: Train-Test Split**

#### The Code:
```python
train_mask = df_model['season'] < 2023
test_mask = df_model['season'] >= 2023

X_train = X[train_mask]
X_test = X[test_mask]
y_train = y[train_mask]
y_test = y[test_mask]
```

#### What this does:
Splits our data into two groups:

**Training Set (2015-2022):**
- Used to **teach** the model
- The model learns patterns from this data
- 4,024 player-seasons

**Test Set (2023):**
- Used to **evaluate** the model
- Data the model has never seen before
- 480 player-seasons
- We use 2023 stats to predict 2024 goals

#### The Output:
```
Training set: 4,024 samples (11.64 avg goals)
Test set: 480 samples (13.42 avg goals)
Training seasons: 2015 - 2022
Test season: 2023 (predicting 2024)
```

#### Why split the data?

**Analogy**: Learning for an exam
- **Training set** = Practice problems you study from
- **Test set** = The actual exam with new questions

If you memorize the exact exam questions beforehand, you're not really testing your understanding. Same with machine learning!

#### Why use time-based split?

We could randomly split the data, but using **time** is smarter for predictions:
- **Real-world scenario**: We only have past data when predicting the future
- **No cheating**: The model can't accidentally learn from future seasons
- **Realistic evaluation**: Tests how well we predict actual future performance

#### Understanding the averages:
- Training average: **11.64 goals** per player-season
- Test average: **13.42 goals** per player-season

The test set has a slightly higher average. This could mean:
1. Players in 2023 were more productive
2. Scoring was up league-wide
3. Our sample of 480 players happened to include more scorers

This difference will make predictions slightly harder (the model learned on lower-scoring data).

---

### **Cell 21: Step 4.4 Title (Markdown)**

Explains we're about to **scale** (standardize) the features.

---

### **Cell 22: Feature Scaling**

#### The Code:
```python
scaler = StandardScaler()

X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

X_train_scaled = pd.DataFrame(X_train_scaled, columns=feature_columns, index=X_train.index)
X_test_scaled = pd.DataFrame(X_test_scaled, columns=feature_columns, index=X_test.index)
```

#### What this does:
**Standardizes** all features to have:
- Mean (average) = 0
- Standard deviation = 1

This puts all features on the same scale.

#### Why is scaling necessary?

**The Problem:**
Our features have very different ranges:
- `games_played`: 1 to 84
- `I_F_goals`: 0 to 69
- `icetime`: 54 to 112,246 seconds
- `Shooting_Pct`: 0% to 100%

**Why this matters:**
Linear regression treats all features equally. Without scaling, features with bigger numbers dominate the model, even if they're less important.

**Analogy**:
Imagine measuring a room using:
- Length in kilometers (0.005 km)
- Width in millimeters (5,000 mm)

The millimeters seem more important just because the numbers are bigger!

#### What standardization does:

**Before scaling:**
```
games_played: 82
I_F_goals: 33
icetime: 93739
```

**After scaling:**
```
games_played: 1.25   (1.25 standard deviations above average)
I_F_goals: 2.14      (2.14 standard deviations above average)
icetime: 1.63        (1.63 standard deviations above average)
```

Now all features are comparable!

#### The Output:
```
Features scaled successfully

Scaled training data sample:
   games_played  I_F_goals  I_F_primaryAssists  ...
0     -1.799572  -1.119801           -1.036575  ...
```

#### Reading scaled values:
- **Negative values**: Below average
- **Zero**: Exactly average
- **Positive values**: Above average
- **-1.8**: About 1.8 standard deviations below average (quite low)

#### Important detail:
```python
scaler.fit_transform(X_train)  # Learn the scaling from training data
scaler.transform(X_test)        # Apply same scaling to test data
```

We **only** learn the scaling parameters from training data, then apply them to test data. This prevents data leakage.

---

### **Cell 23: Step 5 Title (Markdown)**

We're now ready to train our machine learning model!

---

### **Cell 24: Train the Linear Regression Model**

#### The Code:
```python
model = LinearRegression()
model.fit(X_train_scaled, y_train)

print("Linear Regression Model trained successfully")
print(f"\nModel Intercept: {model.intercept_:.2f}")
print(f"Number of coefficients: {len(model.coef_)}")
```

#### What this does:

**Step 1: Create the model**
```python
model = LinearRegression()
```
Creates an empty linear regression model (like a blank brain ready to learn).

**Step 2: Train the model**
```python
model.fit(X_train_scaled, y_train)
```
This is where the **magic happens**! The model learns the relationship between features (X) and goals (y).

#### What is Linear Regression?

**Simple explanation:**
Linear regression finds the best straight line (or in our case, a multi-dimensional plane) that fits the data.

**The Formula:**
```
Next_Season_Goals = Intercept + (Coef₁ × Feature₁) + (Coef₂ × Feature₂) + ... + (Coef₁₁ × Feature₁₁)
```

**Example (simplified with 2 features):**
```
Next_Season_Goals = 11.64 + (3.5 × Current_Goals) + (2.1 × Shooting_Pct)
```

If a player scored 20 goals with 12% shooting:
```
Predicted = 11.64 + (3.5 × 20) + (2.1 × 12)
         = 11.64 + 70 + 25.2
         = 106.84 goals (unrealistic because we have 11 features balancing each other)
```

#### The Output:
```
Linear Regression Model trained successfully

Model Intercept: 11.64
Number of coefficients: 11
```

#### Understanding the output:

**Intercept (11.64):**
- The baseline prediction when all features are at their average (remember, scaled features average to 0)
- Makes sense: The average player in our training data scored 11.64 goals

**11 Coefficients:**
- One coefficient (weight) for each of our 11 features
- Tells us how much each feature contributes to the prediction
- Positive coefficient = feature increases predicted goals
- Negative coefficient = feature decreases predicted goals

#### What just happened?

The model examined all 4,024 training examples and found the mathematical formula that best predicts goals based on the 11 features. It's like finding the pattern in past data.

---

### **Cell 25: Step 5.1 Title (Markdown)**

Now we'll examine which features are most important to the model.

---

### **Cell 26: Feature Importance Analysis**

#### The Code:
```python
coefficients = pd.DataFrame({
    'Feature': feature_columns,
    'Coefficient': model.coef_
}).sort_values(by='Coefficient', ascending=False)

print("Feature Importance (Coefficients):")
print(coefficients)

# [Visualization code creates a bar chart]
```

#### What this does:
1. Extracts the coefficient (importance weight) for each feature
2. Sorts them from most positive to most negative
3. Creates a horizontal bar chart showing importance

#### The Output (Table):
```
                   Feature  Coefficient
6         Goals_Per_Game     3.824691
0          games_played     2.156581
1             I_F_goals     1.984459
10   Ice_Time_Per_Game     1.745032
...
7          Shooting_Pct    -0.537204
```

#### Understanding Coefficients:

**High Positive Coefficients** (Strong predictors of MORE goals):
1. **Goals_Per_Game (3.82)**: Most important feature!
   - Makes sense: Players who consistently score will likely continue
   - A player with high GPG this year will probably have high GPG next year

2. **games_played (2.16)**: Second most important
   - More games = more opportunities to score
   - Availability/health is crucial

3. **I_F_goals (1.98)**: Current goals matter
   - Past performance predicts future performance
   - A 40-goal scorer will likely score more than a 10-goal scorer next year

4. **Ice_Time_Per_Game (1.75)**: Playing time matters
   - Coaches give ice time to productive players
   - More ice time = more scoring opportunities

**Negative Coefficients** (Inverse relationship):
- **Shooting_Pct (-0.54)**: Slightly negative!
  - Why? High shooting percentage often regresses to average (unsustainable)
  - A player shooting 20% one year might drop to 12% next year
  - This is called "regression to the mean"

#### The Visualization:
A horizontal bar chart showing:
- **Long bars to the right**: Strong positive predictors
- **Long bars to the left**: Strong negative predictors
- **Red dashed line at zero**: The dividing line

#### What this tells us:
The model believes **consistency metrics** (Goals Per Game, Ice Time Per Game) are better predictors than **shooting efficiency**. This makes sense because:
- Efficiency can fluctuate (luck, small sample size)
- Opportunity (games played, ice time) is more stable

---

### **Cell 27: Step 6 Title (Markdown)**

We're now evaluating the model - testing how well it predicts!

---

### **Cell 28: Make Predictions**

#### The Code:
```python
y_pred = model.predict(X_test_scaled)

comparison = pd.DataFrame({
    'Player': df_model[test_mask]['name'].values,
    'Team': df_model[test_mask]['team'].values,
    'Actual_Goals': y_test.values,
    'Predicted_Goals': y_pred.round(1)
})
```

#### What this does:
1. **Makes predictions** for all 480 players in the test set (2023 season)
2. Creates a comparison table showing:
   - Player name and team
   - **Actual goals** (what they really scored in 2024)
   - **Predicted goals** (what our model thought they'd score)

#### The Output:
```
Predictions vs Actual (Sample):

               Player Team  Actual_Goals  Predicted_Goals
0          A.J. Greer  CGY           6.0              3.3
1        Adam Edstrom  NYR           5.0              2.5
2       Adam Fantilli  CBJ          31.0             13.7
3       Adam Henrique  EDM          12.0             16.9
...
14      Alex Ovechkin  WSH          44.0             29.1
```

#### Reading the results:

**Good Predictions:**
- **Adam Henrique**: Predicted 16.9, Actual 12.0 (off by 4.9 goals)
  - Pretty close! The model slightly overestimated

**Problematic Predictions:**
- **Adam Fantilli**: Predicted 13.7, Actual 31.0 (off by 17.3 goals!)
  - The model badly underestimated
  - Why? Probably a breakout season the model didn't see coming (young player improvement)

- **Alex Ovechkin**: Predicted 29.1, Actual 44.0 (off by 14.9 goals)
  - Underestimated the Great 8!
  - Elite players sometimes defy statistical predictions

- **A.J. Greer**: Predicted 3.3, Actual 6.0 (off by 2.7 goals)
  - Not bad for a low-scoring player

#### What we learn:
- Model does reasonably well for average players
- Struggles with outliers (breakouts, superstars)
- Young players with limited history are hard to predict

---

### **Cell 29: Step 6.2 Title (Markdown)**

We're creating our first evaluation visualization.

---

### **Cell 30: Predicted vs Actual Scatter Plot**

#### The Code:
```python
plt.scatter(y_test, y_pred, alpha=0.6, color='blue', edgecolors='black', s=50)
plt.plot([min_val, max_val], [min_val, max_val], '--', color='red', linewidth=2, label='Perfect Prediction')
```

#### What this does:
Creates a **scatter plot** comparing predictions to reality.

- **X-axis**: Actual goals scored (truth)
- **Y-axis**: Predicted goals (our model's guess)
- **Red dashed line**: Perfect prediction line (if predicted = actual)
- **Blue dots**: Each dot is one player

#### The Visualization Explained:

**Perfect Model:**
All dots would sit exactly on the red line.

**What we actually see:**
- **Dots close to line**: Good predictions
- **Dots above line**: Model underestimated (predicted too low)
- **Dots below line**: Model overestimated (predicted too high)
- **Spread/scatter**: Prediction error

#### The Output:
```
Interpretation:
- Points close to the red line indicate accurate predictions
- Points above the line = model underestimated goals
- Points below the line = model overestimated goals
```

#### Reading the plot:

**Good signs:**
- Most dots cluster around the red line
- The overall trend follows the line (positive correlation)

**Warning signs:**
- Some dots far from the line (big errors)
- More scatter at higher goal totals (harder to predict elite scorers)
- Few predictions above 35 goals (model conservative with high predictions)

#### What this tells us:
The model is **generally accurate** but **conservative** - it tends to predict closer to average rather than extreme values. This is common in linear regression.

---

### **Cell 31: Step 6.3 Title (Markdown)**

We're creating a residual plot to analyze prediction errors.

---

### **Cell 32: Residual Plot**

#### The Code:
```python
residuals = y_test - y_pred

plt.scatter(y_pred, residuals, alpha=0.6, color='purple', edgecolors='black', s=50)
plt.axhline(y=0, linestyle='--', color='red', linewidth=2)
```

#### What this does:
Creates a **residual plot** that shows prediction errors.

**Residual** = Actual - Predicted
- **Positive residual**: We predicted too low (underestimated)
- **Negative residual**: We predicted too high (overestimated)
- **Zero residual**: Perfect prediction

#### What we're looking for:

**Good residual plot:**
- Random scatter around the red line (y=0)
- No patterns or trends
- Equal spread above and below zero

**Bad residual plot:**
- Curved patterns (model missing non-linear relationships)
- Funnel shape (predictions worse for certain ranges)
- Most residuals on one side (systematic bias)

#### The Output:
```
Interpretation:
- Random scatter around 0 = good model fit
- Patterns or trends = model may be missing important relationships
- Larger residuals at higher predictions = model struggles with high scorers
```

#### What we actually see:

**Positive signs:**
- Mostly random scatter (good!)
- Residuals centered around zero (no major bias)

**Areas for improvement:**
- Larger spread at higher predictions (20+ goals)
  - This means the model is less confident with elite scorers
- Some very large positive residuals (big underestimates)
  - The model missed some breakout performances

#### Why this matters:
The residual plot helps us understand **where** and **why** the model makes mistakes, not just how big the mistakes are.

---

### **Cell 33: Step 6.4 Title (Markdown)**

Now we'll calculate numerical metrics to quantify model performance.

---

### **Cell 34: Calculate Error Metrics**

#### The Code:
```python
mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)
```

#### What this does:
Calculates four key performance metrics.

#### The Output:
```
MODEL PERFORMANCE METRICS
==================================================
Mean Absolute Error (MAE):      5.14 goals
Root Mean Squared Error (RMSE): 6.67 goals
Mean Squared Error (MSE):       44.46
R² Score:                       0.6224
==================================================

Interpretation:
- On average, predictions are off by 5.1 goals
- The model explains 62.2% of the variation in next season goals
- Moderate predictive performance (R² between 0.5-0.7)
```

#### Understanding Each Metric:

---

**1. MAE (Mean Absolute Error) = 5.14 goals**

**Formula:** Average of |Actual - Predicted|

**Example:**
```
Player 1: |30 - 25| = 5
Player 2: |10 - 15| = 5
Player 3: |20 - 19| = 1
Average: (5 + 5 + 1) / 3 = 3.67
```

**What 5.14 means:**
On average, our predictions are off by about **5 goals** (could be 5 too high or 5 too low).

**Is this good?**
- For a player predicted to score 20 goals: 5-goal error is 25% off
- For a player predicted to score 10 goals: 5-goal error is 50% off
- **Moderate performance** - usable but not perfect

---

**2. RMSE (Root Mean Squared Error) = 6.67 goals**

**Formula:** √(Average of (Actual - Predicted)²)

**Why square the errors?**
Penalizes large errors more heavily.

**Example:**
```
Error of 10 goals: 10² = 100
Error of 2 goals:  2² = 4
```
A 10-goal error is treated as much worse than five 2-goal errors.

**What 6.67 means:**
Average error is 6.67 goals, with big errors weighing heavily.

**RMSE vs MAE:**
- RMSE (6.67) > MAE (5.14)
- This gap tells us we have some **large outlier errors**
- If RMSE ≈ MAE, errors would be consistent
- If RMSE >> MAE, we have occasional huge mistakes

---

**3. MSE (Mean Squared Error) = 44.46**

**Formula:** Average of (Actual - Predicted)²

**What it means:**
MSE is just RMSE before taking the square root. It's harder to interpret (units are "goals squared"), so we focus on RMSE.

**Why report it?**
Some technical comparisons use MSE, but RMSE is more intuitive.

---

**4. R² Score = 0.6224 (62.24%)**

**Formula:** 1 - (Sum of squared errors / Total variance)

**What it means:**
The model explains **62.24%** of the variation in next season's goals.

**Analogy:**
Imagine predicting test scores:
- **R² = 1.0 (100%)**: Perfect prediction - explains everything
- **R² = 0.6 (60%)**: Explains 60% of why scores vary; 40% is other factors
- **R² = 0 (0%)**: Model no better than guessing the average

**What 0.6224 means:**
- **62.24%** of goal variation is explained by our 11 features
- **37.76%** is due to other factors:
  - Luck (bounces, post hits)
  - Unmeasured factors (linemates, coaching, injuries)
  - Randomness in sports

**Is 0.62 good?**
- **R² > 0.7**: Strong model
- **R² = 0.5-0.7**: **Moderate model** ← We're here
- **R² < 0.5**: Weak model

For sports predictions, 0.62 is quite respectable! Hockey has lots of randomness.

---

#### Summary of Performance:

**Strengths:**
- Explains majority (62%) of goal variation
- Average error (5.1 goals) is reasonable for hockey

**Weaknesses:**
- Still 38% unexplained (inherent unpredictability)
- Some large outlier errors (RMSE > MAE)
- Room for improvement

**Real-world usefulness:**
This model is **good enough for general forecasting** but shouldn't be the only factor in major decisions (like $50M contracts).

---

### **Cell 35: Step 7 Title (Markdown)**

Now we'll dive deeper into where the model succeeds and fails.

---

### **Cell 36: Best and Worst Predictions**

#### The Code:
```python
comparison['Error'] = comparison['Actual_Goals'] - comparison['Predicted_Goals']
comparison['Abs_Error'] = abs(comparison['Error'])

print("Top 10 Most Accurate Predictions:")
print(comparison.nsmallest(10, 'Abs_Error')...)

print("Top 10 Largest Prediction Errors:")
print(comparison.nlargest(10, 'Abs_Error')...)
```

#### What this does:
1. Calculates error for each player
2. Finds the 10 best predictions (smallest error)
3. Finds the 10 worst predictions (largest error)

#### The Output - BEST Predictions:
```
Top 10 Most Accurate Predictions:
                 Player Team  Actual_Goals  Predicted_Goals  Error
204        John Beecher  BOS           3.0              3.0    0.0  ← PERFECT!
434         Tomas Tatar  SEA           7.0              7.0    0.0  ← PERFECT!
438        Trevor Lewis  LAK           6.0              6.0    0.0  ← PERFECT!
450          Tyson Jost  BUF           4.0              4.0    0.0  ← PERFECT!
225     Justin Danforth  CBJ           9.0              9.1   -0.1  ← Nearly perfect
```

#### Understanding the best predictions:

**Perfect predictions (0.0 error):**
These players scored **exactly** what the model predicted!

**Why were these so accurate?**
Looking at the players:
- Most are **middle-of-the-pack performers** (3-9 goals)
- Consistent players with stable roles
- No major changes in their situations
- The model is best at predicting "average" outcomes

**Key insight:**
The model excels at predicting players who maintain consistent performance.

---

#### The Output - WORST Predictions:
```
Top 10 Largest Prediction Errors:
              Player Team  Actual_Goals  Predicted_Goals  Error
364  Pavel Dorofeyev  VGK          35.0             11.9   23.1  ← Brutal miss!
22   Aliaksei Protas  WSH          30.0              8.5   21.5  ← Breakout season
323    Morgan Geekie  BOS          33.0             12.3   20.7  ← Underestimated
131   Dylan Holloway  EDM          26.0              6.1   19.9  ← Young player surge
253   Leon Draisaitl  EDM          52.0             32.9   19.1  ← Elite scorer
283    Mathew Barzal  NYI           6.0             23.4  -17.4  ← OVERESTIMATED!
```

#### Understanding the worst predictions:

**Massive underestimates (positive errors):**

1. **Pavel Dorofeyev (VGK)**: Predicted 11.9, Actual 35.0
   - **Error: +23.1 goals**
   - Why? Likely a breakout season - young player with limited history
   - Model couldn't predict his development

2. **Aliaksei Protas (WSH)**: Predicted 8.5, Actual 30.0
   - **Error: +21.5 goals**
   - Another breakout player
   - Probably got promoted to top line, boosting opportunities

3. **Leon Draisaitl (EDM)**: Predicted 32.9, Actual 52.0
   - **Error: +19.1 goals**
   - Elite player who exceeded even high expectations
   - Model conservative with extreme predictions

**Massive overestimate (negative error):**

**Mathew Barzal (NYI)**: Predicted 23.4, Actual 6.0
- **Error: -17.4 goals**
- The only major overestimate in top 10!
- What happened?
  - Injury that limited production?
  - Team struggles?
  - Role change?
- This shows the model can't predict unexpected declines

---

#### Key Patterns in Prediction Errors:

**Model underestimates (can't predict):**
- Breakout seasons (young players improving)
- Elite performances beyond historical norms
- Players getting significantly more ice time/opportunities

**Model overestimates (can't predict):**
- Injuries or health issues
- Team context changes (bad linemates)
- Regression after career years
- Reduced roles

**Model gets right:**
- Consistent, established players
- Average performers (10-20 goal range)
- Players in stable situations

---

### **Cell 37: Step 7.2 Title (Markdown)**

Analyzing where the model is strong vs. weak based on goal-scoring ranges.

---

### **Cell 38: Performance by Goal Range**

#### The Code:
```python
comparison['Goal_Range'] = pd.cut(comparison['Actual_Goals'],
                                   bins=[0, 10, 20, 30, 100],
                                   labels=['0-10', '11-20', '21-30', '30+'])

performance_by_range = comparison.groupby('Goal_Range').agg({
    'Abs_Error': ['mean', 'std', 'count']
})
```

#### What this does:
1. **Groups players** into categories based on actual goals scored:
   - **0-10 goals**: Low scorers (grinders, fourth-liners, rookies)
   - **11-20 goals**: Middle scorers (solid contributors)
   - **21-30 goals**: Good scorers (second-line players)
   - **30+ goals**: Elite scorers (stars)

2. **Calculates statistics** for each group:
   - **Mean error**: Average prediction error in that range
   - **Std error**: How consistent the errors are
   - **Count**: How many players in each group

#### The Output (Table):
```
Model Performance by Goal Range:
            Mean_Error  Std_Error  Count
Goal_Range
0-10              3.65       3.01    281
11-20             5.16       3.72    127
21-30             8.01       4.93     50
30+              11.86       6.45     22
```

#### Reading the results:

**0-10 Goals (Low scorers): 281 players**
- **Mean error: 3.65 goals**
- **Best performance!** The model is most accurate here
- Why?
  - Largest sample size (281 players = most training data)
  - Less variance - these players consistently score low
  - Easier to predict "they'll score 5-8 goals"

**11-20 Goals (Middle scorers): 127 players**
- **Mean error: 5.16 goals**
- Good performance, slightly worse than low scorers
- Still reasonable accuracy

**21-30 Goals (Good scorers): 50 players**
- **Mean error: 8.01 goals**
- **Performance declining**
- Harder to predict - more variance in this range
- Smaller sample size = less training data

**30+ Goals (Elite scorers): 22 players**
- **Mean error: 11.86 goals**
- **Worst performance!**
- Why so bad?
  - **Tiny sample**: Only 22 elite scorers (limited training data)
  - **High variance**: Elite seasons can vary wildly (35 to 65 goals)
  - **Outliers**: Breakout seasons, career years
  - **Model conservatism**: Regression pulls predictions toward average

#### The Visualization:
A bar chart showing mean error increasing as goal totals increase.

**The pattern:**
```
      Error
  12 |                                    ████
  10 |                          ████     ████
   8 |                ████      ████     ████
   6 |     ████       ████      ████     ████
   4 |     ████       ████      ████     ████
   2 |     ████       ████      ████     ████
   0 |_____|__________|_________|________|____
      0-10   11-20     21-30      30+
           Goals Scored (Actual)
```

Clear upward trend = model worse at predicting high scorers.

---

#### What This Tells Us:

**Model strengths:**
- **Reliable for most players** (0-20 goals)
- This covers 408 of 480 players (85%)!
- Good for evaluating depth players and middle-six forwards

**Model weaknesses:**
- **Unreliable for elite scorers** (30+ goals)
- Can't distinguish between 35-goal season and 55-goal season
- Needs more data on elite players or different modeling approach

**Real-world implications:**

**Good use cases:**
- "Should we sign this 10-goal scorer?" → Model trustworthy
- "Is this 15-goal player worth $2M?" → Model helpful

**Bad use cases:**
- "Will Connor McDavid score 50 or 65 goals?" → Model unreliable
- "Is this player's 40-goal season repeatable?" → Model uncertain

**Recommendations:**
For elite players (30+ goals), combine this model with:
- Scouting reports
- Age and career trajectory analysis
- Advanced metrics (expected goals models)
- Comparative analysis with similar elite players

---

### **Cell 39: Step 8 Title (Markdown)**

We're saving our work for future use!

---

### **Cell 40: Save Model and Results**

#### The Code:
```python
joblib.dump(model, 'goals_prediction_model.pkl')
joblib.dump(scaler, 'goals_scaler.pkl')
comparison.to_csv('goals_predictions_2023.csv', index=False)
```

#### What this does:
Saves three files to disk:

**1. goals_prediction_model.pkl**
- The trained linear regression model
- Contains all the learned coefficients and parameters
- **Use case**: Load this model later to make new predictions without retraining

**2. goals_scaler.pkl**
- The StandardScaler with learned mean/std for each feature
- **Use case**: Need this to scale new data the same way before predicting

**3. goals_predictions_2023.csv**
- CSV file with all predictions vs. actuals
- **Use case**: Further analysis in Excel, sharing results, creating reports

#### The Output:
```
Model and results saved successfully:
- goals_prediction_model.pkl
- goals_scaler.pkl
- goals_predictions_2023.csv
```

#### Why save the model?

**Without saving:**
Every time you want a prediction, retrain on 4,024 player-seasons (slow!).

**With saving:**
```python
# Future use - just 3 lines!
model = joblib.load('goals_prediction_model.pkl')
scaler = joblib.load('goals_scaler.pkl')
predictions = model.predict(scaler.transform(new_player_data))
```

#### Real-world example:

**October 2024**: New season starts
1. Load the saved model
2. Collect first 20 games of stats for all players
3. Scale the new data using the saved scaler
4. Predict rest-of-season goals
5. Update predictions as season progresses

**No retraining needed!**

---

### **Cell 41: Final Insights and Reflection (Markdown)**

This is a comprehensive written analysis summarizing everything learned. Let me break down the key sections:

---

#### **Section 1: Model Performance**

**Summary:**
- R² score shows substantial predictive power
- MAE shows reasonable accuracy
- Better for mid-range scorers than extremes

**Translation:**
The model works well overall but isn't perfect. It's like a B+ student - good but not exceptional.

---

#### **Section 2: Most Important Features**

**Key findings:**
1. **Current season goals**: Best predictor
   - Makes intuitive sense - past predicts future
2. **Goals per game**: Normalization helps
3. **Expected goals (xG)**: Shot quality matters
4. **High-danger shots**: Quality chances lead to goals

**Why this matters:**
These features tell us **what actually drives goal scoring**:
- Consistency (GPG)
- Opportunity (games played, ice time)
- Skill (xG, shot quality)

---

#### **Section 3: Model Strengths**

**What it does well:**
- Established players with consistent playing time
- 10-30 goal range (the majority)
- Captures shot volume + quality relationship

**Translation:**
If you're a regular NHL forward with a stable role, the model will predict your performance pretty accurately.

---

#### **Section 4: Model Weaknesses**

**What it struggles with:**

**1. Extreme outliers**
- Breakout seasons (player suddenly improves)
- Major slumps (player suddenly declines)

**2. External factors it can't see:**
- **Injuries**: No way to predict who gets hurt
- **Line changes**: Getting promoted to play with McDavid changes everything
- **Coaching changes**: New systems affect production
- **Contract year motivation**: Some players elevate in contract years
- **Age**: Young players improve, old players decline

**3. Limited historical data**
- Young players: Only 1-2 seasons of history
- The model needs several years to identify patterns

**4. Elite scorers**
- Tends to underestimate 40+ goal scorers
- Can't predict 60-goal seasons

**Why these weaknesses exist:**
The model only knows **statistics**. It doesn't know:
- Player X is recovering from injury
- Player Y just got traded to a better team
- Player Z is motivated for a contract
- Player W is aging and slowing down

---

#### **Section 5: Practical Applications**

**Where this model IS useful:**

**Contract negotiations:**
- "Player wants $5M for 3 years. Will he produce?"
- Model predicts 18 goals → helps set fair price

**Trade analysis:**
- "Team wants 2 first-round picks for this 25-goal scorer"
- Model predicts 12 goals next year → maybe he's overvalued

**Lineup decisions:**
- "Who should get top-line minutes?"
- Model helps identify consistent producers

**Prospect evaluation:**
- Track young players year-over-year
- Are they progressing as expected?

**Fantasy hockey:**
- Draft players model predicts will improve
- Trade away players model predicts will decline

---

**Where this model is NOT useful (alone):**

**Major contract decisions:**
- Don't offer $50M based solely on this model
- Combine with scouting, medical, character evaluation

**Elite player evaluation:**
- Model underestimates stars
- Need specialized approaches for superstars

**Context-specific predictions:**
- Player changing teams? Model doesn't know
- Player recovering from injury? Model doesn't know
- Player in contract year? Model doesn't know

---

#### **Section 6: Reflection**

**Key learnings:**

**1. Feature engineering matters**
- Goals Per Game > raw goals
- Ratios and rates > absolute totals
- Domain knowledge (understanding hockey) helps create better features

**2. Time-based splitting is crucial**
- Can't use future data to predict future (circular logic)
- Simulates real-world forecasting

**3. Linear models have limitations**
- Can't capture complex non-linear patterns
- Example: Age has a curved relationship (improve when young, decline when old)
- Linear regression treats it as straight line

**4. Domain knowledge enhances modeling**
- Understanding hockey helped:
  - Select relevant features
  - Engineer meaningful metrics
  - Interpret results correctly
  - Identify model weaknesses

---

**Challenges encountered:**

**1. Incomplete data**
- Players traded mid-season (split stats between teams)
- Injuries (some seasons cut short)
- Different numbers of games played

**2. Complexity vs. interpretability**
- Could use Random Forest, Neural Networks (more accurate)
- Chose Linear Regression (more interpretable)
- Trade-off: Understand why vs. slightly better predictions

**3. Data leakage risk**
- Easy to accidentally use next season's data
- Required careful feature selection
- Validation: Does this feature exist before predicting?

---

**Future improvements:**

**1. Include age**
- Young players (21-24): Likely to improve
- Prime players (25-29): Likely stable
- Older players (30+): Might decline
- Could add age as feature or create age-based models

**2. Team-level statistics**
- Offensive system (high vs. low scoring teams)
- Power play opportunities
- Quality of linemates

**3. Ensemble methods**
- Random Forest: Captures non-linear patterns
- Gradient Boosting: Often more accurate
- Trade-off: Less interpretable

**4. Separate models by player type**
- Snipers (high shooting %, fewer shots)
- Volume shooters (low shooting %, many shots)
- Playmakers (low goals, high assists)
- Different types might need different models

**5. Rolling averages**
- Instead of single-season stats
- Use 3-year averages to smooth variance
- Reduces impact of outlier seasons

---

#### **Final Conclusion:**

**Summary statement:**
> "This linear regression model provides a solid foundation for goal prediction and demonstrates that statistical modeling can meaningfully support hockey analytics and team decision-making."

**Translation:**
The model isn't perfect, but it's **useful**. It won't replace scouts and coaches, but it can **inform** their decisions with data-driven insights.

**The broader lesson:**
Machine learning is a **tool**, not a magic solution. It:
- Helps quantify patterns humans might miss
- Provides objective baselines for expectations
- Works best when combined with expert judgment
- Has limitations that must be understood

**Real-world application:**
Modern NHL teams use models like this as **one input** among many:
- Analytics team: Provides statistical predictions
- Scouts: Evaluate eye test, intangibles
- Coaches: Assess fit with system
- Medical: Evaluate health and durability
- Management: Combines all inputs for decisions

This notebook demonstrates the complete workflow that analytics departments use every day!

---

## Key Concepts Explained

### What is Machine Learning?

**Simple definition:**
Teaching computers to learn patterns from data without explicitly programming every rule.

**Traditional programming:**
```
If player scored 30+ goals AND shoots >12% THEN predict 25 goals
If player scored 10-20 goals AND shoots <10% THEN predict 12 goals
...
```
You write every rule manually.

**Machine learning:**
```
Here's 4,024 examples of player-seasons and their next-year goals.
Computer: Find the patterns yourself!
```
The computer discovers the rules from data.

---

### What is Linear Regression?

**Concept:**
Finding the best straight line (or multi-dimensional plane) that fits your data.

**Formula:**
```
y = b₀ + b₁x₁ + b₂x₂ + ... + bₙxₙ
```

Where:
- **y**: What we're predicting (Next Season Goals)
- **b₀**: Intercept (baseline value)
- **b₁, b₂, ..., bₙ**: Coefficients (weights for each feature)
- **x₁, x₂, ..., xₙ**: Features (Current goals, GPG, etc.)

**Visual analogy (2D):**
Imagine plotting points on a graph and drawing the line that gets closest to all points.

---

### What is Training and Testing?

**Training:**
The learning phase. Model examines training data and adjusts its coefficients to minimize errors.

**Testing:**
The evaluation phase. Model makes predictions on new data it's never seen to measure real-world performance.

**Why separate?**
Prevents overfitting (memorizing training data instead of learning general patterns).

---

### What are Features?

**Features** = Input variables used for prediction

**Types:**
- **Raw features**: Directly from data (goals, assists, shots)
- **Engineered features**: Calculated from raw features (Goals Per Game, Shooting %)

**Good features:**
- Predictive (correlate with target)
- Available before prediction (no future data)
- Independent (not redundant)

---

### What is Standardization/Scaling?

**Problem:**
Features have different units and ranges:
- Ice time: 1,000-100,000 seconds
- Goals: 0-70

**Solution:**
Transform all features to same scale (mean=0, std=1).

**Benefit:**
Model treats all features fairly regardless of original units.

---

### What are Evaluation Metrics?

**MAE (Mean Absolute Error):**
- Average size of errors
- Easy to understand ("off by 5 goals on average")
- Treats all errors equally

**RMSE (Root Mean Squared Error):**
- Penalizes large errors more
- RMSE > MAE means you have some big mistakes

**R² Score (R-squared):**
- Percentage of variance explained
- 0 = model useless, 1 = perfect model
- 0.62 = explains 62% of why goals vary

---

## Summary of Results

### Overall Performance
- **R² Score**: 0.6224 (62.24% variance explained)
- **MAE**: 5.14 goals average error
- **RMSE**: 6.67 goals (accounting for larger errors)

### Best Performance
- **Low scorers (0-10 goals)**: 3.65 goal average error
- **Middle scorers (11-20 goals)**: 5.16 goal average error
- Covers 85% of players with reasonable accuracy

### Worst Performance
- **Elite scorers (30+ goals)**: 11.86 goal average error
- Only 22 players in this category
- Model tends to underestimate breakouts and superstars

### Most Important Features
1. Goals Per Game (3.82 coefficient)
2. Games Played (2.16 coefficient)
3. Current Season Goals (1.98 coefficient)
4. Ice Time Per Game (1.75 coefficient)

### Key Insights
- **Past performance is the best predictor** of future performance
- **Opportunity metrics** (games played, ice time) matter more than efficiency (shooting %)
- **Consistency matters** - the model works best for established, stable players
- **Context matters** - the model can't account for injuries, trades, or other external factors

### Practical Value
✅ **Good for**: General forecasting, depth player evaluation, establishing baselines
❌ **Not good for**: Elite player predictions, accounting for context changes, contract-year players

---

## Final Takeaway

This notebook successfully demonstrates a complete machine learning workflow for sports analytics. While the model has limitations (as all models do), it provides **meaningful, data-driven insights** that can support decision-making when combined with expert judgment and contextual understanding.

The 62% R² score shows that **statistics can predict a significant portion** of future performance, but the remaining 38% reminds us that **hockey (and sports in general) always has an element of unpredictability** - and that's what makes it exciting!

---

**Document created by: AI Assistant**
**For: Assignment 5 - NHL Goals Prediction Model**
**Purpose: Detailed explanation for students learning ML and data analysis**
**Date: November 3, 2025**
