# 🎮 GTA 6 — Day 1 Revenue Prediction

<div align="center">

![Made with Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-Workbench-4479A1?style=flat-square&logo=mysql&logoColor=white)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)
![Chart.js](https://img.shields.io/badge/Chart.js-Dashboard-FF6384?style=flat-square&logo=chart.js&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen?style=flat-square)

**An end-to-end machine learning pipeline predicting GTA 6's opening day revenue**
**using 44 historical AAA game launches as training data.**

### 🔴 [→ View Live Interactive Dashboard](https://doelnk.github.io/gta6-revenue-prediction)

<br/>

| 🎯 Predicted Revenue | 📊 Games Analyzed | 📅 Data Range | 🏆 vs. Industry Avg |
|:---:|:---:|:---:|:---:|
| **$1,890M** | **44 titles** | **2010 – 2024** | **7.4×** |

</div>

---

## 💡 Why I Built This

I'm an aspiring data analyst who loves video games — and I've been learning SQL and Python on the side. When GTA 6 was announced, I had one thought:

> *"This game is going to break every record ever set. But by how much exactly?"*

That question became this project.

Instead of just guessing, I decided to actually build something. I wanted to collect real data on historical blockbuster game launches, build a proper relational database, run a machine learning model, and see what the numbers say. Two goals drove this from day one: **learn by doing**, and **predict something I genuinely care about**.

I picked GTA 6 specifically because the data tells a compelling story — a $1 billion development budget, 210 million trailer views, a franchise with 5 entries of compounding brand loyalty, and a fanbase that has been waiting over a decade. If any game was going to produce an interesting prediction, it was this one.

This project taught me more SQL, Python, and data thinking in a few weeks than months of tutorials ever did. It turns out the best way to learn is to work on something you actually want to answer.

---

## 🗺️ Project Journey

This wasn't a straight line. Here's honestly how it went:

**1. Started with a question, built a database**
I designed a 4-table relational schema in MySQL Workbench from scratch — normalized, with primary and foreign keys — and manually researched and entered data for 44 AAA game launches going back to 2010.

**2. Hit data quality problems immediately**
Some revenue figures I found online were week-1 numbers disguised as day-1 figures (Fallout 4 being the biggest culprit). Some games launched on Game Pass day one, making their revenue structurally incomparable to premium launches. I had to research, flag, and in some cases correct every row.

**3. Built the Python pipeline**
I loaded the data into Pandas, engineered 5 calculated feature columns, ran a multicollinearity check (found that `release_year` and `cpi_inflation_index` were nearly identical signals — dropped `release_year`), and trained a Linear Regression model in scikit-learn.

**4. Started with Power BI — then switched**
My original plan was to build the final dashboard in Power BI. I got it working — dark theme, bar chart, KPI card. But I ran into a real limitation: Power BI is desktop software, it requires a Windows install for the full experience, and the web version on Mac has gaps. More importantly, I couldn't share it as a simple link.

I realized an interactive HTML dashboard would be more accessible — anyone with a browser can open it, no login, no software, no friction. So I worked with Claude to transform my Power BI layout into a fully interactive HTML dashboard using Chart.js. The design decisions from Power BI (color scheme, chart types, KPI layout) carried over, but now it lives at a URL anyone can visit.

**5. Iterated on the data more than the model**
The biggest lesson: the model is only as good as the data. I spent far more time fixing data quality issues than tuning the model — and that's probably the most realistic data analyst experience I could have had.

---

## 🏗️ Pipeline Architecture

```
┌─────────────────┐     ┌──────────────────────┐     ┌─────────────────────┐
│  MySQL          │     │  Python / VS Code     │     │  HTML Dashboard     │
│  Workbench      │ ──► │                       │ ──► │                     │
│                 │     │  Pandas               │     │  Chart.js           │
│  4 tables       │     │  Feature engineering  │     │  5 interactive      │
│  Primary keys   │     │  scikit-learn         │     │  charts             │
│  Foreign keys   │     │  LinearRegression     │     │  4 filter modes     │
│  JOIN query     │     │  model.predict()      │     │  Live on GitHub     │
│  CSV export     │     │  CSV output           │     │  Pages              │
└─────────────────┘     └──────────────────────┘     └─────────────────────┘
```

---

## 🗄️ The Database

I designed a normalized relational schema with 4 tables linked by `game_id`:

```sql
historical_launches    -- Core: title, publisher, release year, dev budget, day 1 revenue
macro_economics        -- Financial: retail price, CPI inflation index at launch
pre_release_buzz       -- Hype: trailer views, trailer likes, Google Trends score
console_market_size    -- Hardware: active PS + Xbox install base at launch (millions)
```

The master extraction query that feeds Python:

```sql
SELECT
    hl.game_title,
    hl.release_year,
    hl.development_budget_millions,
    hl.day_1_revenue_millions,
    me.base_retail_price,
    me.cpi_inflation_index,
    pb.trailer_1_views_millions,
    pb.trailer_1_likes_millions,
    pb.google_trends_score,
    cm.active_consoles_millions
FROM historical_launches hl
INNER JOIN macro_economics   me ON hl.game_id = me.game_id
INNER JOIN pre_release_buzz  pb ON hl.game_id = pb.game_id
INNER JOIN console_market_size cm ON hl.game_id = cm.game_id;
```

---

## ⚙️ Feature Engineering

Five calculated columns built on top of the raw data to expose relationships the raw numbers hide:

```python
# 1. CPI-adjusted revenue — normalizes cross-era comparability
#    A 2013 dollar is not a 2024 dollar. This was the most important fix.
df['real_revenue_millions'] = df['day_1_revenue_millions'] * (142.50 / df['cpi_inflation_index'])

# 2. Hype efficiency — trailer reach relative to what was spent making the game
df['hype_per_budget'] = df['trailer_1_views_millions'] / df['development_budget_millions']

# 3. Engagement quality — filters viral inflation from genuine enthusiasm
#    Raw views can be misleading. Like/view ratio is harder to fake.
df['like_view_ratio'] = df['trailer_1_likes_millions'] / df['trailer_1_views_millions']

# 4. Market penetration intensity
df['budget_per_console'] = df['development_budget_millions'] / df['active_consoles_millions']

# 5. Price era flag — the $60→$70 shift is a structural break, not a continuous variable
df['is_next_gen_price'] = (df['base_retail_price'] >= 69.99).astype(int)
```

---

## 🤖 The Model

```python
from sklearn.linear_model import LinearRegression

# Exclude Game Pass launches — structurally incomparable to $80 premium releases
# Halo Infinite ($50M day 1) and CoD Black Ops 6 would corrupt the coefficients
train_df = df[
    (df['gamepass_launch'] == 0) &
    (df['is_prediction'] == 0)
]

X_train = train_df[feature_columns]      # Shape: (42, 13)
y_train = train_df['real_revenue_millions']

model = LinearRegression()
model.fit(X_train, y_train)

X_gta6 = df[df['is_prediction'] == 1][feature_columns]
prediction = model.predict(X_gta6)
# → $1,890M
```

**13 features used:**
`development_budget_millions` · `cpi_inflation_index` · `base_retail_price` · `trailer_1_views_millions` · `google_trends_score` · `active_consoles_millions` · `metacritic_score` · `franchise_installment_number` · `pc_day1_launch` · `hype_per_budget` · `like_view_ratio` · `budget_per_console` · `is_next_gen_price`

---

## 🚧 Problems I Hit (And How I Fixed Them)

These are the real problems — not the clean tutorial version.

| Problem | What Happened | How I Fixed It |
|---|---|---|
| **Inflated revenue figures** | Fallout 4 listed as $750M day 1 — actually week 1 | Researched primary sources, corrected to $400M |
| **Game Pass contamination** | Halo Infinite's $50M would teach the model that $200M budget = $50M revenue | Added `gamepass_launch` binary flag, excluded from training |
| **Multicollinearity** | `release_year` and `cpi_inflation_index` were nearly identical signals | Dropped `release_year`, kept CPI as the superior economic signal |
| **Franchise number skew** | FIFA 22 was counting as installment 29, CoD as 17 — distorting the franchise feature | Switched to sub-series installment numbers (FIFA 22 = 3rd modern era FIFA) |
| **Cross-era comparability** | A 2011 game's $400M isn't the same as a 2023 game's $400M | Used CPI-adjusted revenue as the model target, not raw revenue |
| **Exclusive vs. multiplatform install base** | Halo 5 showed 20M consoles — same year as Fallout 4 at 110M | Applied consistent methodology: combined PS+Xbox for multiplatform, platform-specific for exclusives |
| **Power BI sharing limitations** | Couldn't share dashboard as a simple link on Mac | Rebuilt as standalone HTML using Chart.js — works in any browser |

---

## 📊 Dashboard Features

**[→ Open the live dashboard](https://doelnk.github.io/gta6-revenue-prediction)**

| Visual | What It Shows |
|---|---|
| **4 KPI cards** | Predicted revenue · Historical average · Industry multiplier · Best comp |
| **Bar chart** | Top 15 launches + GTA 6, color-coded by revenue tier |
| **Scatter plot** | Budget vs. revenue — dot size = Metacritic score, GTA 6 as outlier |
| **ROI table** | Revenue-to-budget multiplier rankings — who delivered the best return |
| **Confidence chart** | Data quality distribution: HIGH / MEDIUM / LOW |
| **CPI comparison** | Raw vs. inflation-adjusted revenue for top 5 games |

**4 live filter buttons** — all charts update simultaneously:
- All 44 games
- High confidence sources only
- Exclude Game Pass launches
- Rockstar peers only

---

## 📁 Repository Structure

```
gta6-revenue-prediction/
│
├── index.html                   ← Live interactive dashboard
├── README.md                    ← You are here
│
├── data/
│   └── power_bi_master.csv      ← Final dataset (45 rows, 25 columns)
│
├── sql/
│   └── master_extraction.sql    ← MySQL schema + JOIN query
│
└── python/
    ├── feature_engineering.py   ← 5 engineered column calculations
    ├── model_training.py        ← LinearRegression training + prediction
    └── correlation_heatmap.py   ← Multicollinearity analysis
```

---

## 🔑 Key Findings

- GTA 6's **210M trailer views** are 3.2× the next closest game in the dataset (Black Myth: Wukong at 65M) — and trailer views are one of the strongest revenue predictors in the model
- When adjusted for inflation, **GTA V's $800M in 2013 = $1,140M in 2026 dollars** — making it the closest true financial peer to GTA 6
- **Game Pass launches** (Halo Infinite: $50M, CoD BO6: $500M flagged) are statistical outliers — including them in training would have suppressed the prediction by an estimated 12–18%
- The $10 **price increase** from $60 to $70/$80 is a structural break in the data — treated as a binary era flag rather than a continuous variable
- GTA 6 is predicted to outperform GTA V's raw day 1 record by **136%**

---

## ⚠️ Limitations (Being Honest)

- **42 training rows** — workable, but below the ideal 10–20 samples per feature for 13 features
- **Linear assumption** — revenue relationships may be non-linear in ways this model doesn't capture
- **Unknown factors** — actual Metacritic score (projected 95), server stability on launch day, and regional pricing can't be modeled in advance
- Many revenue figures are **estimated from week 1 data** — all flagged with a `data_confidence` column (HIGH / MEDIUM / LOW)

---

## 🚀 What's Next

- [ ] Ridge / Lasso Regression to handle remaining multicollinearity with regularization
- [ ] IGDB API integration — automated data ingestion to replace manual research
- [ ] Bootstrap confidence intervals — produce a prediction range, not just a point estimate
- [ ] Residual plot — visualize where the model over/under-fits across training data
- [ ] Update with actual GTA 6 day 1 revenue when it launches 🎮

---

## 🛠️ Tech Stack

| Tool | Use |
|---|---|
| MySQL Workbench | Relational schema design, data storage, JOIN queries |
| Python 3 + Pandas | Data ingestion, cleaning, feature engineering |
| scikit-learn | Linear Regression model training and prediction |
| Seaborn | Correlation heatmap for multicollinearity analysis |
| Chart.js | Interactive dashboard charts |
| HTML / CSS / JS | Dashboard frontend |
| GitHub Pages | Free hosting for the live dashboard |

---

<div align="center">

*Built by someone who loves data, loves video games, and wanted to answer a question that actually mattered to them.*

*Pipeline: MySQL → Python / scikit-learn → Interactive HTML Dashboard*

*Data: Publisher press releases · IGDB API · VGChartz · US BLS CPI-U · Metacritic*

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/doel-nkalo-b15385263/)
[![Portfolio](https://img.shields.io/badge/Portfolio-Visit-FF6384?style=flat-square&logo=github&logoColor=white)](https://doelnk.github.io)

</div>
