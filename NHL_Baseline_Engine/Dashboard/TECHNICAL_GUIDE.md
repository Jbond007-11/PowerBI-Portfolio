# Technical Implementation Guide

**Project:** Baseline Expectation Engine (NHL Performance Variance)

---

## Phase 1: The Prediction Engine (Python Prerequisites)

Before opening Power BI, the baseline data was generated in Python.

### Process Overview

1. **Data Collection:** Sourced historical NHL forward data (2015-2023).

2. **Feature Engineering:** Calculated Goals Per Game, Shot Quality, and Ice Time Efficiency.

3. **Training:** Trained a Linear Regression model (sklearn) to predict goal totals.

4. **Export:** Saved the output as `goals_predictions_2023.csv`.

---

## Phase 2: Data Engineering (Power Query / ETL)

This phase merged the static Python predictions with live API data inside Power BI.

### Step 1: Connect to the NHL API (Live Data)

* Created a custom M Function (`fxGetNHLAPI`) to fetch data from `https://api.nhle.com/stats/rest/en/skater/summary`.

* Used a pagination strategy (List of pages 0–20) to retrieve the full league roster (~800 players).

* Expanded the JSON records to create the table `NHL_Stats`.

### Step 2: Import the Predictions (Static Data)

* Imported `goals_predictions_2023.csv` as a new query named `ML_Predictions`.

* Promoted headers to ensure columns were named correctly (`Player`, `Predicted_Goals`).

### Step 3: Data Normalization

To ensure the Merge worked despite capitalization differences, a custom Join Key was created in both tables:

* **Formula:** `Text.Upper(Text.Trim([Player]))`

* **Result:** "CONNOR MCDAVID" (Standardized).

### Step 4: The "Filter Join"

Performed a Merge Queries action starting from `NHL_Stats`:

* Joined with `ML_Predictions` using the `JoinKey`.

* **Join Type:** Inner Join.

* **Why:** This automatically filtered out Defensemen, Goalies, and Rookies who existed in the API but not in the Machine Learning model.

* Expanded the `ML_Predictions` column to bring in `Predicted_Goals`.

* Renamed `Predicted_Goals` to `Baseline Projection`.

---

## Phase 3: Analytical Logic (DAX)

Created measures to quantify the gap between expectation and reality.

### Measure: Performance Delta

Calculates the variance.

```dax
Performance Delta = SUM('NHL_Stats'[Actual Goals]) - SUM('NHL_Stats'[Baseline Projection])
```

**Interpretation:**

* **Positive (+)** = Overperforming (Breakout).

* **Negative (-)** = Underperforming (Risk).

---

## Phase 4: Dashboard Visualization

Designed the "Executive View" for General Managers.

### Visual 1: The Performance Matrix (Scatter Plot)

* **X-Axis:** Baseline Projection (The Model).

* **Y-Axis:** Actual Goals (Live Data).

* **Values:** Player.

* **Analytics Line:** Added a dashed Trend Line (Unity Line).
  * Dots above: Good contracts.
  * Dots below: Bad contracts.

* **Conditional Formatting:**
  * **Color Style:** Gradient based on Performance Delta.
  * **Scale:** Red (Low) → Grey (Mid) → Green (High).

### Visual 2: Variance Leaders (Bar Charts)

Created two Clustered Bar Charts to list specific names.

#### A. Top 5 Breakouts (Green Chart)

* **X-Axis:** Performance Delta.

* **Y-Axis:** Player.

* **Filter Logic:** Used "Filters on this visual" → Player → Top N (5) → By Value: Performance Delta.

#### B. Top 5 Risks (Red Chart)

* **X-Axis:** Performance Delta.

* **Y-Axis:** Player.

* **Filter Logic:** Used "Filters on this visual" → Player → Bottom N (5) → By Value: Performance Delta.

### Visual 3: Contextual Slicers

* **Team Slicer:** Allows filtering by specific NHL franchise.

* **Position Slicer:** Allows comparison of Centers vs. Wingers.

---

## Phase 5: Polish & UX

* **Background:** Added a hockey-themed background image with transparency to set context.

* **Titles:** Changed technical labels (e.g., "Sum of Delta") to business terms (e.g., "Overperformance").

* **Subtitle:** Added a text box explanation of the model logic to build user trust.

---

## Tech Stack Summary

* **Python (Backend):** Pandas, Scikit-Learn (Linear Regression Model).

* **Power BI (Frontend):** Power Query (M), DAX, Data Visualization.

* **Data Sources:** Official NHL API (Live Data) & Historical CSV logs (Training Data).

---

## Key Insights

**The Goal:** Identify contract inefficiencies (Breakout Stars vs. Regression Risks) in real-time.

NHL General Managers often fall into the trap of **"Recency Bias"**—overpaying players based on a single lucky season, or trading away players who are statistically due for a bounce-back.

This project solves that problem by building a **Baseline Expectation Engine** that uses historical data to predict what a player *should* score based on underlying play-driving metrics, and compares it live against what they *are* scoring.
