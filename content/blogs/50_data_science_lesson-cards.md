---
title: "50 Data Science Lesson Cards"
date: 2026-03-13
draft: false
---


<style>
*{box-sizing:border-box;margin:0;padding:0}
body{background:transparent}
.wrap{padding:16px 0;font-family:'Courier New',monospace}
.top{display:flex;align-items:center;justify-content:space-between;margin-bottom:16px;flex-wrap:wrap;gap:8px}
.filters{display:flex;gap:6px;flex-wrap:wrap}
.fb{padding:4px 12px;border-radius:20px;border:1px solid var(--color-border-tertiary);background:var(--color-background-secondary);color:var(--color-text-secondary);cursor:pointer;font-size:11px;font-family:inherit}
.fb.on{background:var(--color-background-info);color:var(--color-text-info);border-color:var(--color-border-info)}
.count{font-size:12px;color:var(--color-text-tertiary)}
.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(280px,1fr));gap:12px}
.card{background:#0d1117;border-radius:12px;padding:20px;cursor:pointer;border:1px solid #21262d;transition:border-color .15s,transform .15s;position:relative;min-height:180px;display:flex;flex-direction:column;gap:10px}
.card:hover{border-color:#388bfd;transform:translateY(-2px)}
.card-top{display:flex;align-items:center;justify-content:space-between}
.tag{padding:3px 9px;border-radius:20px;font-size:10px;font-weight:700;letter-spacing:.5px}
.num{font-size:11px;color:#484f58}
.title{font-size:14px;font-weight:700;color:#e6edf3;line-height:1.4}
.body{font-size:12px;color:#8b949e;line-height:1.6;flex:1}
.code{background:#010409;border-radius:6px;padding:8px 10px;font-size:11px;color:#79c0ff;line-height:1.6;border-left:2px solid #388bfd;white-space:pre;overflow-x:auto}
.tip{font-size:11px;color:#3fb950;margin-top:auto;padding-top:8px;border-top:1px solid #21262d}

.overlay{position:fixed;inset:0;background:rgba(0,0,0,.85);display:flex;align-items:center;justify-content:center;z-index:100;padding:20px}
.modal{background:#0d1117;border:1px solid #30363d;border-radius:16px;padding:32px;max-width:520px;width:100%;position:relative;max-height:90vh;overflow-y:auto}
.modal-tag{display:inline-block;padding:5px 14px;border-radius:20px;font-size:12px;font-weight:700;margin-bottom:14px}
.modal-num{font-size:12px;color:#484f58;margin-bottom:6px}
.modal-title{font-size:22px;font-weight:900;color:#e6edf3;line-height:1.3;margin-bottom:14px}
.modal-body{font-size:14px;color:#8b949e;line-height:1.8;margin-bottom:16px}
.modal-code{background:#010409;border-radius:8px;padding:14px;font-size:13px;color:#79c0ff;line-height:1.7;border-left:3px solid #388bfd;white-space:pre-wrap;margin-bottom:14px}
.modal-tip{font-size:13px;color:#3fb950;padding:10px 14px;background:#0d2113;border-radius:8px;line-height:1.6}
.modal-nav{display:flex;justify-content:space-between;align-items:center;margin-top:20px;padding-top:16px;border-top:1px solid #21262d}
.mnav-btn{padding:7px 16px;background:#21262d;color:#8b949e;border:1px solid #30363d;border-radius:8px;cursor:pointer;font-size:12px;font-family:inherit}
.mnav-btn:hover{background:#30363d;color:#e6edf3}
.close{position:absolute;top:14px;right:14px;background:transparent;border:none;color:#484f58;cursor:pointer;font-size:18px;line-height:1}
.close:hover{color:#e6edf3}
.share-note{font-size:11px;color:#484f58;text-align:center;margin-top:8px}
</style>

<div class="wrap" id="app"></div>

<script>
const CARDS = [
  {id:1,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"groupby goes on the DataFrame, not the column",body:"A very common mistake: calling .groupby() on a column Series instead of the DataFrame itself.",code:"# WRONG\ndf['sales'].groupby('region').sum()\n\n# RIGHT\ndf.groupby('region')['sales'].sum()",tip:"Think: DataFrame → group → select → aggregate. Always left to right."},
  {id:2,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"Boolean filters need & not 'and'",body:"Python's 'and' is scalar. Pandas needs element-wise & for boolean array operations.",code:"# WRONG — TypeError\ndf[df['sales']>100 and df['region']=='N']\n\n# RIGHT\ndf[(df['sales']>100) & (df['region']=='N')]",tip:"Each condition must be in its own parentheses — & has higher precedence than >."},
  {id:3,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:".dt is the gateway to all datetime parts",body:"Once a column is datetime dtype, you ALWAYS access date parts through the .dt accessor.",code:"df['date'] = pd.to_datetime(df['date'])\n\ndf['month']   = df['date'].dt.month\ndf['year']    = df['date'].dt.year\ndf['weekday'] = df['date'].dt.dayofweek",tip:"These are properties, not methods. Use .dt.month not .dt.month()"},
  {id:4,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"fit() on train only — transform() on both",body:"Fitting a scaler on test data leaks test statistics into training. The scaler must learn only from training data.",code:"scaler = StandardScaler()\n\n# RIGHT\nX_train = scaler.fit_transform(X_train)\nX_test  = scaler.transform(X_test)  # no fit!\n\n# WRONG\nX_test = scaler.fit_transform(X_test)",tip:"This is one of the most common causes of data leakage in ML pipelines."},
  {id:5,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"transform() keeps the original shape",body:"Unlike agg(), transform() broadcasts the group result back to every row — same index, same length.",code:"# Add each row's region mean as a new column\ndf['region_avg'] = df.groupby('region')['sales']\\\n                     .transform('mean')\n\n# Now subtract it (no merge needed)\ndf['vs_avg'] = df['sales'] - df['region_avg']",tip:"If you need to add a group-level stat back to every row, transform() is always the answer."},
  {id:6,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"melt() converts wide to long format",body:"Wide format has one column per variable. Long format has one row per observation. melt() reshapes wide → long.",code:"df.melt(\n  id_vars='product',\n  var_name='quarter',\n  value_name='revenue'\n)\n# Turns Q1, Q2, Q3, Q4 columns\n# into a single 'quarter' column",tip:"Long format is what most ML models and ggplot-style charts expect."},
  {id:7,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"left join keeps all rows from the left table",body:"The 4 join types: inner (intersection), left (all left), right (all right), outer (union).",code:"# Keep ALL orders, even without a customer match\norders.merge(customers,\n             on='customer_id',\n             how='left')\n# Missing customer → NaN, row stays",tip:"Default merge() is inner join — it silently drops unmatched rows. Always specify how=."},
  {id:8,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:".cumsum() for running totals within groups",body:"Sort your data first — cumsum() is order-dependent. The result changes completely if rows are unsorted.",code:"df = df.sort_values(['region', 'date'])\n\ndf['running_total'] = (\n  df.groupby('region')['sales']\n    .cumsum()\n)",tip:"A missing sort_values() before cumsum() is a silent bug — no error, wrong answer."},
  {id:9,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"np.select() is the clean alternative to nested apply",body:"apply() with a lambda is slow and verbose for conditional logic. np.select() is vectorized and readable.",code:"conditions = [\n  df['score'] >= 90,\n  df['score'] >= 75,\n  df['score'] >= 60,\n]\nchoices = ['A', 'B', 'C']\ndf['grade'] = np.select(conditions, choices, default='F')",tip:"Conditions are evaluated in order. First True match wins."},
  {id:10,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"drop_duplicates() has subset= and keep= params",body:"By default it checks all columns. Use subset= to check specific ones, keep= to control which duplicate survives.",code:"# Keep the LAST purchase per customer\ndf.drop_duplicates(\n  subset='customer_id',\n  keep='last'\n)",tip:"keep=False drops ALL duplicates — useful when you want only truly unique rows."},
  {id:11,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"rolling() needs a window, then an aggregation",body:".rolling() creates a sliding window object. You still need to call .mean(), .sum(), .std() etc. on it.",code:"# 7-day rolling average\ndf['rolling_avg'] = (\n  df['sales'].rolling(window=7).mean()\n)\n# First 6 values will be NaN",tip:"Add min_periods=1 to avoid NaNs at the start: rolling(7, min_periods=1)."},
  {id:12,topic:"Pandas",color:"#1f6feb",bg:"#0d1f3c",title:"fillna() with mean is the baseline imputation strategy",body:"Always compute the mean BEFORE filling — and fit on train data only to avoid leakage.",code:"# Fill with column mean\nmean_val = df['score'].mean()\ndf['score'] = df['score'].fillna(mean_val)\n\n# Or inline\ndf['score'].fillna(df['score'].mean(), inplace=True)",tip:"More advanced: fill with group median, use KNNImputer, or flag missingness as a feature."},

  {id:13,topic:"SQL",color:"#238636",bg:"#0d2113",title:"HAVING filters aggregates — WHERE cannot",body:"WHERE runs before GROUP BY. HAVING runs after. You cannot use SUM(), COUNT() etc. in a WHERE clause.",code:"-- WRONG: WHERE SUM(amount) > 5000\n-- RIGHT:\nSELECT rep_id, SUM(amount) total\nFROM sales\nGROUP BY rep_id\nHAVING SUM(amount) > 5000",tip:"Rule: if your filter references an aggregate function, it's always HAVING."},
  {id:14,topic:"SQL",color:"#238636",bg:"#0d2113",title:"Window functions don't collapse rows",body:"GROUP BY collapses many rows into one. Window functions compute across rows and KEEP all rows.",code:"SELECT sale_id, amount,\n  SUM(amount) OVER (\n    ORDER BY sale_date\n  ) AS running_total\nFROM sales\n-- All rows preserved",tip:"OVER() is what makes it a window function. An empty OVER() applies the function across all rows."},
  {id:15,topic:"SQL",color:"#238636",bg:"#0d2113",title:"PARTITION BY resets the window per group",body:"Think of PARTITION BY as GROUP BY for window functions — it creates separate windows per group.",code:"SELECT *,\n  RANK() OVER (\n    PARTITION BY category\n    ORDER BY revenue DESC\n  ) AS rank_in_category\nFROM products\n-- Rank resets for each category",tip:"Without PARTITION BY, you get a single ranking across all rows."},
  {id:16,topic:"SQL",color:"#238636",bg:"#0d2113",title:"RANK vs DENSE_RANK vs ROW_NUMBER",body:"Given scores 100, 100, 90 — the three functions give different results for ties.",code:"ROW_NUMBER → 1, 2, 3  (always unique)\nRANK       → 1, 1, 3  (skips 2)\nDENSE_RANK → 1, 1, 2  (no gaps)\n\n-- Use RANK() for sports-style rankings\n-- Use DENSE_RANK() when gaps are confusing",tip:"ROW_NUMBER is useful for deduplication: delete rows WHERE row_num > 1."},
  {id:17,topic:"SQL",color:"#238636",bg:"#0d2113",title:"Self join: join a table to itself with aliases",body:"The classic puzzle — find employees who earn more than their manager. One table, two roles.",code:"SELECT e.name, e.salary\nFROM employees e\nJOIN employees m\n  ON e.manager_id = m.employee_id\nWHERE e.salary > m.salary",tip:"Alias the table twice (e = employee, m = manager) then join on the relationship."},
  {id:18,topic:"SQL",color:"#238636",bg:"#0d2113",title:"Filter window functions in an outer query",body:"You cannot use WHERE to filter on a window function result in the same query level — wrap in a subquery.",code:"SELECT * FROM (\n  SELECT *,\n    RANK() OVER (\n      PARTITION BY category\n      ORDER BY revenue DESC\n    ) AS rnk\n  FROM products\n) ranked\nWHERE rnk <= 3",tip:"CTEs (WITH clauses) are cleaner than subqueries for this pattern."},
  {id:19,topic:"SQL",color:"#238636",bg:"#0d2113",title:"LIMIT 2 ≠ top 2 per group",body:"LIMIT gives you the top N overall. For top N per group, you need a window function + subquery.",code:"-- WRONG: top 2 products total\nSELECT * FROM products\nORDER BY revenue DESC LIMIT 2\n\n-- RIGHT: top 2 per category\n-- Use RANK() OVER (PARTITION BY...)",tip:"This is the most commonly failed SQL interview question."},
  {id:20,topic:"SQL",color:"#238636",bg:"#0d2113",title:"CTEs make complex queries readable",body:"A CTE (WITH clause) is a named temporary result set. Use them to break complex queries into readable steps.",code:"WITH high_earners AS (\n  SELECT rep_id, SUM(amount) total\n  FROM sales\n  GROUP BY rep_id\n  HAVING SUM(amount) > 5000\n)\nSELECT * FROM high_earners\nJOIN reps USING (rep_id)",tip:"Multiple CTEs are separated by commas. Each can reference the previous one."},
  {id:21,topic:"SQL",color:"#238636",bg:"#0d2113",title:"NULL is not equal to anything — including NULL",body:"NULL = NULL is not TRUE in SQL. Use IS NULL or IS NOT NULL. This trips up every beginner.",code:"-- WRONG: returns nothing\nSELECT * FROM users WHERE email = NULL\n\n-- RIGHT\nSELECT * FROM users WHERE email IS NULL\n\n-- COALESCE replaces NULL with a default\nSELECT COALESCE(email, 'unknown')",tip:"COUNT(*) counts all rows. COUNT(column) skips NULLs. A subtle but critical difference."},
  {id:22,topic:"SQL",color:"#238636",bg:"#0d2113",title:"LAG() and LEAD() access previous/next row values",body:"Two of the most useful window functions — compare a row to its predecessor or successor.",code:"SELECT sale_date, amount,\n  LAG(amount, 1) OVER (\n    ORDER BY sale_date\n  ) AS prev_amount,\n  amount - LAG(amount,1) OVER (\n    ORDER BY sale_date\n  ) AS day_change\nFROM sales",tip:"LAG(col, N) looks back N rows. LEAD(col, N) looks forward N rows."},
  {id:23,topic:"SQL",color:"#238636",bg:"#0d2113",title:"CASE WHEN is SQL's if/else",body:"Use CASE WHEN for conditional logic directly in SELECT, ORDER BY, or aggregations.",code:"SELECT name,\n  CASE\n    WHEN salary > 100000 THEN 'Senior'\n    WHEN salary > 60000  THEN 'Mid'\n    ELSE 'Junior'\n  END AS level\nFROM employees",tip:"CASE WHEN inside SUM() is a powerful pattern: SUM(CASE WHEN ... THEN 1 ELSE 0 END)."},
  {id:24,topic:"SQL",color:"#238636",bg:"#0d2113",title:"EXPLAIN shows you the query execution plan",body:"Before optimizing a slow query, run EXPLAIN (or EXPLAIN ANALYZE) to see what the database is actually doing.",code:"EXPLAIN SELECT * FROM orders\nWHERE customer_id = 42\n\n-- Look for: Seq Scan (bad on large tables)\n-- Want: Index Scan\n-- Fix: CREATE INDEX ON orders(customer_id)",tip:"Sequential scan on millions of rows is usually the culprit for slow queries."},

  {id:25,topic:"Python",color:"#d29922",bg:"#271d04",title:"Counter is your best friend for frequency counts",body:"Don't write loops to count frequencies. collections.Counter does it in one line and sorts by frequency.",code:"from collections import Counter\n\nwords = ['cat','dog','cat','bird','dog','cat']\nc = Counter(words)\n\nc.most_common(2)\n# [('cat', 3), ('dog', 2)]",tip:"Counter inherits from dict — all dict methods work on it."},
  {id:26,topic:"Python",color:"#d29922",bg:"#271d04",title:"enumerate() replaces range(len(...))",body:"range(len(x)) is a code smell. enumerate() gives you both index and value cleanly.",code:"# UGLY\nfor i in range(len(names)):\n    print(i, names[i])\n\n# CLEAN\nfor i, name in enumerate(names):\n    print(i, name)\n\n# Start from 1\nfor i, name in enumerate(names, start=1):",tip:"enumerate() is one of the most underused Python builtins."},
  {id:27,topic:"Python",color:"#d29922",bg:"#271d04",title:"zip() iterates multiple lists in parallel",body:"zip() pairs up elements from multiple iterables. Stops at the shortest one.",code:"names  = ['Alice', 'Bob', 'Carol']\nscores = [95, 82, 78]\n\nfor name, score in zip(names, scores):\n    print(f'{name}: {score}')\n\n# Unzip with: names, scores = zip(*pairs)",tip:"Use itertools.zip_longest() if you want to continue past the shortest list."},
  {id:28,topic:"Python",color:"#d29922",bg:"#271d04",title:"Two Sum: hashmap gives O(n), nested loops give O(n²)",body:"For each element, store its index. Then check if (target - element) is already stored. One pass.",code:"def two_sum(nums, target):\n    seen = {}\n    for i, n in enumerate(nums):\n        complement = target - n\n        if complement in seen:\n            return [seen[complement], i]\n        seen[n] = i",tip:"The hashmap lookup is O(1) — making the whole function O(n) instead of O(n²)."},
  {id:29,topic:"Python",color:"#d29922",bg:"#271d04",title:"List comprehensions are faster than for loops",body:"List comprehensions use optimized C-level iteration. They're typically 35–50% faster than equivalent for loops.",code:"# SLOW\nresult = []\nfor x in data:\n    if x > 0:\n        result.append(x * 2)\n\n# FAST\nresult = [x*2 for x in data if x > 0]",tip:"Dict and set comprehensions work the same way: {k:v for k,v in d.items()}"},
  {id:30,topic:"Python",color:"#d29922",bg:"#271d04",title:"defaultdict removes the 'key exists?' check",body:"Stop writing if key in dict — defaultdict automatically creates a default value on first access.",code:"from collections import defaultdict\n\n# Grouping without key-exists check\ngroups = defaultdict(list)\nfor item in data:\n    groups[item['category']].append(item)\n\n# Counter equivalent with defaultdict(int)",tip:"defaultdict(list), defaultdict(int), defaultdict(set) cover 90% of use cases."},
  {id:31,topic:"Python",color:"#d29922",bg:"#271d04",title:"IQR rule: outliers live beyond 1.5× the interquartile range",body:"Q1 = 25th percentile, Q3 = 75th percentile. IQR = Q3 - Q1. This is the same rule matplotlib boxplots use.",code:"import numpy as np\n\nq1, q3 = np.percentile(data, [25, 75])\niqr = q3 - q1\nlower = q1 - 1.5 * iqr\nupper = q3 + 1.5 * iqr\n\noutliers = [x for x in data\n            if x < lower or x > upper]",tip:"More robust than mean ± 3σ when distributions are skewed."},
  {id:32,topic:"Python",color:"#d29922",bg:"#271d04",title:"*args and **kwargs make functions flexible",body:"*args captures extra positional arguments as a tuple. **kwargs captures extra keyword arguments as a dict.",code:"def log(msg, *args, **kwargs):\n    level    = kwargs.get('level', 'INFO')\n    print(f'[{level}] {msg}', *args)\n\nlog('User logged in', 'alice', level='DEBUG')",tip:"The names args/kwargs are convention only — *data and **options work equally well."},
  {id:33,topic:"Python",color:"#d29922",bg:"#271d04",title:"Generators are lazy — they compute on demand",body:"A generator yields one value at a time instead of building the entire list in memory. Essential for large data.",code:"# List: builds all 1M items in memory\nbig_list = [x**2 for x in range(1_000_000)]\n\n# Generator: computes one at a time\nbig_gen = (x**2 for x in range(1_000_000))\n\n# Same syntax with () instead of []",tip:"Use generators when you process data once and don't need random access."},
  {id:34,topic:"Python",color:"#d29922",bg:"#271d04",title:"@lru_cache memoizes expensive function calls",body:"Caches results of pure function calls. If called again with the same arguments, returns the cached result instantly.",code:"from functools import lru_cache\n\n@lru_cache(maxsize=None)\ndef fibonacci(n):\n    if n < 2:\n        return n\n    return fibonacci(n-1) + fibonacci(n-2)\n\n# Recursive fib without cache: O(2ⁿ)\n# With cache: O(n)",tip:"Only use on pure functions (same inputs always give same outputs, no side effects)."},
  {id:35,topic:"Python",color:"#d29922",bg:"#271d04",title:"dataclasses reduce boilerplate for data containers",body:"Instead of writing __init__, __repr__, __eq__ by hand — use @dataclass. One decorator, all the boilerplate gone.",code:"from dataclasses import dataclass\n\n@dataclass\nclass Product:\n    name: str\n    price: float\n    in_stock: bool = True\n\np = Product('Widget', 9.99)\nprint(p)  # Product(name='Widget'...)",tip:"Add frozen=True to make instances immutable (hashable, usable as dict keys)."},
  {id:36,topic:"Python",color:"#d29922",bg:"#271d04",title:"Moving average: the slice is data[i-window+1 : i+1]",body:"The hardest part is getting the slice indices right. For a window of size n ending at position i:",code:"def moving_avg(data, window):\n    result = []\n    for i in range(len(data)):\n        if i < window - 1:\n            result.append(None)  # incomplete window\n        else:\n            w = data[i - window + 1 : i + 1]\n            result.append(sum(w) / window)\n    return result",tip:"Pad the start with None (or NaN) — don't compute partial windows unless intended."},

  {id:37,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"Accuracy is meaningless on imbalanced datasets",body:"A model that predicts 'not fraud' for every transaction gets 99.9% accuracy but catches zero fraud.",code:"# Better metrics for imbalanced data:\n\nPrecision = TP / (TP + FP)\nRecall    = TP / (TP + FN)\nF1        = 2 * P * R / (P + R)\n\n# Best: AUC-PR (precision-recall curve)\n# Not: AUC-ROC (can hide poor minority perf)",tip:"Always ask: what's the cost of a false positive vs a false negative in this domain?"},
  {id:38,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"Data leakage: the #1 cause of suspiciously good results",body:"Leakage happens when information from outside the training window contaminates the model. Result: great on paper, terrible in production.",code:"# Classic leakage example:\n# Predicting hospital readmission using\n# 'discharge_notes' — only written AFTER\n# readmission decision was made.\n\n# Ask: would I have this feature\n# at prediction time?",tip:"If your model looks too good to be true, check for leakage before anything else."},
  {id:39,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"The bias-variance tradeoff in plain English",body:"Bias = your model makes wrong assumptions (too simple). Variance = your model memorizes noise (too complex).",code:"Total Error = Bias² + Variance + Noise\n\nHigh bias (underfitting):\n  Simple model, wrong on everything\n\nHigh variance (overfitting):\n  Complex model, wrong on new data\n\nGoal: find the sweet spot",tip:"Regularization, cross-validation, and ensembles are tools for managing this tradeoff."},
  {id:40,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"AUC-PR > AUC-ROC for imbalanced data",body:"With severe class imbalance, a high AUC-ROC can hide terrible minority class performance. AUC-PR is harder to inflate.",code:"# Model A: AUC-ROC 0.89, AUC-PR 0.43\n# Model B: AUC-ROC 0.84, AUC-PR 0.71\n\n# For fraud detection (0.5% positive):\n# Deploy Model B — it actually finds fraud\n\n# AUC-ROC can look great just by\n# correctly classifying the 99.5%",tip:"Use AUC-PR as your primary metric whenever positives are rare (< 10% of data)."},
  {id:41,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"Always split time series by time — never randomly",body:"Shuffling time series data causes future data to leak into training. Your model sees tomorrow while predicting yesterday.",code:"# WRONG: random shuffle on time series\nX_train, X_test = train_test_split(df)\n\n# RIGHT: temporal split\ntrain = df[df['date'] < '2024-01-01']\ntest  = df[df['date'] >= '2024-01-01']\n\n# CV: use TimeSeriesSplit",tip:"Walk-forward validation mirrors how the model will actually be used in production."},
  {id:42,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"L1 shrinks weights to zero. L2 shrinks them toward zero.",body:"L1 (Lasso) produces sparse models — useful for feature selection. L2 (Ridge) distributes penalty across all weights.",code:"# L1 (Lasso): sum of |weights|\n# Drives some weights exactly to 0\n# Good for: feature selection\n\n# L2 (Ridge): sum of weights²\n# Shrinks all weights, rarely to 0\n# Good for: correlated features\n\n# ElasticNet: combines both",tip:"When in doubt, start with Ridge. Add Lasso when you need interpretability."},
  {id:43,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"Model in production degrading? Check for drift first.",body:"Performance drops without code changes = distribution shift. The real world has changed since you trained the model.",code:"Three types of drift:\n\n1. Covariate shift:\n   Input features changed\n\n2. Label shift:\n   Target distribution changed\n\n3. Concept drift:\n   Feature→target relationship changed",tip:"Monitor feature distributions in production. Alert when they diverge from training distributions."},
  {id:44,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"Random Forest vs Gradient Boosting: when to use which",body:"Both are powerful ensemble methods, but they fail differently and have different speed/accuracy tradeoffs.",code:"Random Forest (bagging):\n  + Parallel training → fast\n  + Robust to noisy data\n  + Less hyperparameter tuning\n\nGradient Boosting (sequential):\n  + Higher accuracy\n  - Slower, more sensitive to params\n  - Overfits noisy data easily",tip:"For tabular data competitions: XGBoost/LightGBM. For production with speed needs: Random Forest."},
  {id:45,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"SHAP gives you per-prediction feature importance",body:"Model.feature_importances_ is global. SHAP gives you local (per-prediction) AND global explanations.",code:"import shap\n\nexplainer = shap.TreeExplainer(model)\nshap_values = explainer.shap_values(X_test)\n\n# Why did this one prediction happen?\nshap.force_plot(explainer.expected_value,\n               shap_values[0], X_test.iloc[0])",tip:"SHAP is game-theory based — it's the most theoretically sound importance method."},
  {id:46,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"Calibration: does 0.8 confidence mean 80% accuracy?",body:"A model predicting P=0.8 for 100 cases should have ~80 actually positive. Miscalibrated models give unreliable probabilities.",code:"# Check calibration:\nfrom sklearn.calibration import CalibrationDisplay\n\nCalibrationDisplay.from_predictions(\n    y_test, y_prob\n)\n# Diagonal = perfect calibration\n\n# Fix: Platt Scaling or\n# Isotonic Regression",tip:"Critical when you're using probabilities to set thresholds or rank customers."},
  {id:47,topic:"ML",color:"#8957e5",bg:"#1a0f2e",title:"The 5 pre-deployment checks every model needs",body:"Before handing off to engineering, run these 5 checks — not just accuracy.",code:"1. Performance on held-out test set\n   (once, at the very end)\n\n2. Calibration check\n\n3. Data leakage audit\n\n4. Slice analysis\n   (performance across subgroups)\n\n5. Training-serving skew check",tip:"#5 is the most often skipped and the most common cause of production failures."},

  {id:48,topic:"Stats",color:"#e85d04",bg:"#2d0f00",title:"p = 0.04 does NOT mean 96% chance the hypothesis is true",body:"The p-value is the probability of observing results this extreme IF the null hypothesis is true. Not the reverse.",code:"p-value = P(data | H₀ is true)\n\nNOT P(H₀ is false | data)\n\nA p < 0.05 only means:\n'If nothing was happening,\nresults this extreme would\noccur < 5% of the time'",tip:"This misinterpretation is so common it has a name: the 'transposed conditional fallacy'."},
  {id:49,topic:"Stats",color:"#e85d04",bg:"#2d0f00",title:"Type I vs Type II errors — and the tradeoff",body:"Lowering α (significance threshold) reduces false positives but increases false negatives. You can't minimize both simultaneously.",code:"Type I  (α): False positive\n  Reject a true null\n  'We saw an effect that isn't there'\n\nType II (β): False negative\n  Miss a real effect\n  'We missed an effect that is there'\n\nPower = 1 - β",tip:"In medicine, Type II errors (missing a disease) are often more costly than Type I."},
  {id:50,topic:"Stats",color:"#e85d04",bg:"#2d0f00",title:"The Central Limit Theorem: why normal distributions are everywhere",body:"Regardless of the population distribution, the sampling distribution of the mean approaches normal as n grows.",code:"CLT in practice:\n\n- Justifies using normal-based\n  confidence intervals\n\n- Works for n ≥ 30 in most cases\n\n- Explains why so many natural\n  phenomena look bell-shaped\n\n- Foundation of hypothesis testing",tip:"The CLT is why you can run a t-test even when your raw data isn't normally distributed."},
];

const TOPIC_COLORS = {
  Pandas:{color:"#388bfd",bg:"rgba(56,139,253,.15)"},
  SQL:{color:"#3fb950",bg:"rgba(63,185,80,.15)"},
  Python:{color:"#e3b341",bg:"rgba(227,179,65,.15)"},
  ML:{color:"#bc8cff",bg:"rgba(188,140,255,.15)"},
  Stats:{color:"#ff7b72",bg:"rgba(255,123,114,.15)"},
};

let filter = "All";
let selected = null;

function tagStyle(topic){
  const t = TOPIC_COLORS[topic];
  return `background:${t.bg};color:${t.color};border:1px solid ${t.color}44`;
}

function renderGrid(){
  const app = document.getElementById('app');
  const shown = filter==="All" ? CARDS : CARDS.filter(c=>c.topic===filter);
  
  let html = `<div class="top">
    <div class="filters">
      ${["All","Pandas","SQL","Python","ML","Stats"].map(t=>`
        <button class="fb ${filter===t?'on':''}" onclick="setFilter('${t}')">${t}</button>
      `).join('')}
    </div>
    <span class="count">${shown.length} cards</span>
  </div>
  <div class="grid">
    ${shown.map(c=>`
      <div class="card" onclick="openCard(${c.id})">
        <div class="card-top">
          <span class="tag" style="${tagStyle(c.topic)}">${c.topic}</span>
          <span class="num">#${String(c.id).padStart(2,'0')}</span>
        </div>
        <div class="title">${c.title}</div>
        <div class="body">${c.body.substring(0,90)}${c.body.length>90?'…':''}</div>
        <div class="code">${c.code.split('\n').slice(0,3).join('\n')}${c.code.split('\n').length>3?'\n…':''}</div>
      </div>
    `).join('')}
  </div>`;

  if(selected){
    const c = selected;
    const idx = CARDS.indexOf(c);
    const tc = TOPIC_COLORS[c.topic];
    html += `<div class="overlay" onclick="closeCard(event)">
      <div class="modal" onclick="event.stopPropagation()">
        <button class="close" onclick="closeModal()">✕</button>
        <div class="modal-num">#${String(c.id).padStart(2,'0')} of 50</div>
        <div class="modal-tag" style="${tagStyle(c.topic)}">${c.topic}</div>
        <div class="modal-title">${c.title}</div>
        <div class="modal-body">${c.body}</div>
        <div class="modal-code">${c.code}</div>
        <div class="modal-tip">💡 ${c.tip}</div>
        <div class="modal-nav">
          <button class="mnav-btn" onclick="navCard(-1)" ${idx===0?'disabled':''}>← Prev</button>
          <span style="font-size:12px;color:#484f58">${idx+1} / 50</span>
          <button class="mnav-btn" onclick="navCard(1)" ${idx===CARDS.length-1?'disabled':''}>Next →</button>
        </div>
        <div class="share-note">Screenshot this card to share on X or LinkedIn</div>
      </div>
    </div>`;
  }

  app.innerHTML = html;
}

function setFilter(f){ filter=f; renderGrid(); }
function openCard(id){ selected=CARDS.find(c=>c.id===id); renderGrid(); }
function closeModal(){ selected=null; renderGrid(); }
function closeCard(e){ if(e.target.classList.contains('overlay')){ selected=null; renderGrid(); } }
function navCard(dir){
  const idx = CARDS.indexOf(selected);
  const next = CARDS[idx+dir];
  if(next){ selected=next; renderGrid(); }
}

renderGrid();
</script>
