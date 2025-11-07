AHP — World Cup 2026 Favorites
================================

Use the Analytic Hierarchy Process (AHP) to rank national teams for the FIFA World Cup 2026.
All scoring logic and checks are kept in Excel; Python reads the Results sheet, exports a CSV, and generates the chart.

[Chart file in repo: ahp_worldcup2026_barh.png]

--------------------------------

Overview
--------
| Item            | Description                                                                        |
|-----------------|------------------------------------------------------------------------------------|
| Objective       | Transparent, reproducible ranking of World Cup 2026 favorites using AHP.          |
| Stack           | Excel (pairwise, normalization, consistency) + Python (read Results, CSV, plot).  |
| Outputs         | Sorted scores in % per team + horizontal bar chart with in-bar labels.            |
| Reproducibility | requirements.txt (minimal), single Python runner.                                  |

Criteria
--------
| # | Criterion                         | Captures                         | Example inputs            |
|---:|----------------------------------|----------------------------------|---------------------------|
| 1 | Current Strength                  | Overall strength signal          | FIFA/Elo-style rating     |
| 2 | Recent Form                       | Short-term momentum/trend        | Recent W/D/L, form index  |
| 3 | Tactical Quality & Coach Experience | Coaching pedigree, system maturity | Coach tenure, titles      |
| 4 | Squad Depth & Market Value        | Bench strength, injuries, valuation | Market value, rotation depth |
| 5 | World Cup History                 | Historic performance in WC       | Titles, deep runs         |

Note: Weights come from Saaty’s pairwise comparisons and are validated with a consistency check.

Method (AHP)
------------
1) Pairwise comparison of criteria
   - Build an n × n positive reciprocal matrix A = [a_ij].
   - Saaty scale: 1 (equal), 3 (moderate), 5 (strong), 7 (very strong), 9 (extreme);
     2/4/6/8 are intermediates; use reciprocals for “less important”.
   - Reciprocity: a_ji = 1/a_ij, a_ii = 1.

2) Criteria weights (priority vector)
   - Principal-eigenvector method: A · w = λ_max · w with sum(w_i) = 1.
   - Alternatives (common approximations): column-normalization + row averages, or geometric mean per row.

3) Consistency check
   - CI = (λ_max − n) / (n − 1)
   - CR = CI / RI
   - Rule of thumb: CR < 0.10 is acceptable.

   Saaty Random Index (RI):
   | n  | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  |
   |----|-----|-----|-----|-----|-----|-----|-----|-----|-----|-----|
   | RI | 0.0 | 0.0 | 0.58| 0.90| 1.12| 1.24| 1.32| 1.41| 1.45| 1.49|

4) Team scoring (weighted synthesis)
   - Let X = [x_ij] be teams i by criteria j. Normalize columns to X~ (sum-to-1, min-max, or suitable scale).
   - Weighted sum per team:
       s_i = Σ_j ( w_j · x~_ij )
       Score_i(%) = 100 × s_i / Σ_k s_k.
   - Final Results = sorted Score (%) by team (used in the chart).

Repository Structure
--------------------
| Path                              | Purpose                                                                 |
|-----------------------------------|-------------------------------------------------------------------------|
| AHP_WorldCup2026_Favorites.xlsx   | All AHP steps: Dataset, Criteria_Pairwise, Consistency, Decision_Matrix, Results. |
| run_ahp_worldcup.py               | Minimal Python runner (reads Results, exports CSV, plots chart).       |
| ahp_worldcup2026_results.csv      | Exported results (sorted) from Python.                                 |
| ahp_worldcup2026_barh.png         | Horizontal bar chart with in-bar % labels.                             |
| README_AHP_WorldCup2026.md        | Original notes (optional).                                             |
| requirements.txt                  | Minimal dependencies: pandas, matplotlib, openpyxl.                    |
| .gitignore                        | Standard Python/Spyder ignores.                                        |
| README.md                         | This document.                                                         |

How to Run Locally
------------------
1) Install dependencies
   pip install -r requirements.txt

2) Generate CSV + chart
   python run_ahp_worldcup.py

What happens
- Reads AHP_WorldCup2026_Favorites.xlsx → sheet Results (Team, Score (%)).
- Sorts and writes ahp_worldcup2026_results.csv.
- Saves ahp_worldcup2026_barh.png (ready for GitHub/portfolio).

Notes & Assumptions
-------------------
- Single-level AHP (criteria → alternatives).
- Criteria treated as benefit after normalization. If a criterion should be minimized, invert before normalization.
- Consistency check (CR) should be < 0.10.
- Excel is the source of truth; Python is a thin renderer (CSV + chart).

References
----------
- T. L. Saaty (1980). The Analytic Hierarchy Process. McGraw-Hill.
- T. L. Saaty & L. G. Vargas (2012). Models, Methods, Concepts & Applications of the Analytic Hierarchy Process. Springer.

Appendix — Python Runner (same as file)
---------------------------------------
# AHP World Cup 2026 — Read Excel, export CSV, and plot horizontal bar chart
import pandas as pd
import matplotlib.pyplot as plt
from pathlib import Path

EXCEL_PATH = Path("AHP_WorldCup2026_Favorites.xlsx")
OUT_CSV = EXCEL_PATH.with_name("ahp_worldcup2026_results.csv")
OUT_PNG = EXCEL_PATH.with_name("ahp_worldcup2026_barh.png")

TEAM_COL = "Team"
SCORE_COL = "Score (%)"

df = pd.read_excel(EXCEL_PATH, sheet_name="Results")
df_sorted = df.sort_values(by=SCORE_COL, ascending=False).reset_index(drop=True)
df_sorted.to_csv(OUT_CSV, index=False)

fig, ax = plt.subplots(figsize=(9, 6))
bars = ax.barh(df_sorted[TEAM_COL], df_sorted[SCORE_COL])

ax.set_xlabel("Score (%)")
ax.set_title("AHP — World Cup 2026 Favorites (Score %)")
ax.invert_yaxis()
ax.grid(axis="x", alpha=0.2)

values = df_sorted[SCORE_COL].to_numpy()
for rect, v in zip(bars, values):
    ax.text(
        rect.get_x() + rect.get_width() / 2,
        rect.get_y() + rect.get_height() / 2,
        f"{v:.2f}%",
        ha="center", va="center",
        color="white", fontsize=10
    )

plt.tight_layout()
plt.savefig(OUT_PNG, dpi=200)
plt.show()
plt.close()
