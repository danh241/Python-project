# Cross-Industry Financial Performance Analysis with Python

## Project Summary

This project focuses on evaluating and comparing the financial health of companies across three distinct sectors (e.g., Tech, FMCG, Real Estate). By consolidating data from Balance Sheets and Income Statements, this analysis identifies profitability trends, solvency risks, and operational efficiency.

The goal is to build an automated data processing pipeline to screen for top-performing companies based on key financial metrics like **ROE (Return on Equity)** and **Net Profit Margin**.

**Tools Used:** Python (Pandas, Numpy, Seaborn, Matplotlib)

**Skills Demonstrated:** Data Analysis,  Data Wrangling & Integration, Feature Engineering, Business Insight Generation

---

## 1. Situation

A manufacturing company was struggling with inconsistent product quality, leading to increased material waste and potential customer dissatisfaction. The quality assurance (QA) team lacked a data-driven method for monitoring production output in real-time. Machine adjustments were often based on intuition rather than statistical evidence, making it difficult to identify the root cause of quality issues. The primary business goal was to establish a systematic approach to monitor machine performance and ensure all products meet strict dimensional specifications.

---

## 2. Task

My primary objective was to establish a quantitative analysis framework to evaluate corporate financial health and investment potential.

*   **Consolidate Data:** Merge disparate datasets (Balance Sheet and Income Statement) into a unified view for holistic analysis.
*   **Derive Financial KPIs:** Calculate and assess critical profitability and solvency metrics, such as ROE (Return on Equity), ROA, and Debt-to-Equity Ratios.
*   **Benchmark Performance:** Conduct a comparative industry analysis to identify high-growth sectors and screen for the Top 5 performing companies within the target industry.

---

## 3. Action

My analytical process was executed entirely within , focusing on efficiency and readability.

### Step 1: Data Consolidation & Financial Metric Engineering

I began by establishing a unified data foundation using Pandas to merge disparate financial statements and compute key performance indicators (KPIs) for company analysis.

*   **`pd.merge` with inner join**: This was essential to combine the Balance Sheet and Income Statement datasets. By joining on specific keys (`company`, `comp_type`, `Year`), I ensured that every financial ratio was calculated based on perfectly aligned data points for each specific fiscal year and entity.
*   **Handling Division by Zero (`.replace(0, np.nan)`)**: In financial data, denominators like 'Total Revenue' or 'Total Liabilities' can occasionally be zero. I proactively replaced these zeros with NaN (Not a Number) within the calculation formulas. This robust approach prevents program crashes due to mathematical errors and flags data gaps for transparency.
*   **`Calculated Attributes (Feature Engineering)`**: I programmed the logic for critical financial ratios such as ROE, ROA, and Debt-to-Equity. Specifically, I defined Operating Income as a proxy for net income to isolate and analyze the core operational efficiency of the companies, filtering out non-operational noise.

```python
import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

df_balance_sheet = pd.read_excel("data/Balance_Sheet.xlsx")
df_income_statement = pd.read_excel("data/Income_Statement.xlsx")

merged_df = pd.merge(
    df_balance_sheet,
    df_income_statement,
    on=['company', 'comp_type', 'Year'],
    how='inner'
)

print(merged_df.isnull().sum())

num_duplicates = merged_df.duplicated().sum()
print(num_duplicates)

merged_df.info()

net_income_proxy = 'Operating Income' 

merged_df['Net_Profit_Margin'] = merged_df[net_income_proxy] / merged_df['Total Revenue'].replace(0, np.nan)

merged_df['ROE'] = merged_df[net_income_proxy] / merged_df['Total Stockholder Equity'].replace(0, np.nan)

merged_df['ROA'] = merged_df[net_income_proxy] / merged_df['Total Assets'].replace(0, np.nan)

merged_df['Debt_to_Equity'] = merged_df['Total Liab'] / merged_df['Total Stockholder Equity'].replace(0, np.nan)

merged_df['Current_Ratio'] = merged_df['Total Current Assets'] / merged_df['Total Current Liabilities'].replace(0, np.nan)

final_view = merged_df[[
    'company', 
    'Year',
    'comp_type',
    'Net_Profit_Margin',
    'ROE',
    'ROA',
    'Debt_to_Equity',
    'Current_Ratio'
]]

final_view.round(2).head(10) 
```

### Step 2: Comparative Industry Analysis & Visualization

After computing the individual ratios, I proceeded to aggregate the data to derive industry-level insights, utilizing Python's visualization libraries to uncover patterns in profitability and risk distribution.

*   **`groupby()` aggregation**: I segmented the dataset by industry sector (comp_type) to calculate the mean values for key metrics like ROE and Debt-to-Equity. This pivots the analysis from granular company details to broader sectoral benchmarks, establishing a baseline for performance.
*   **`sns.barplot` for Comparison**: I utilized a Seaborn bar chart to visualize the average Return on Equity (ROE) across different industries. This visualization provides an immediate, high-level ranking of which sectors are delivering the highest returns to shareholders on average.
*   **`sns.boxplot` for Distribution Analysis**: Recognizing that averages can be misleading, I implemented a box plot to examine the spread of ROE within each industry. This reveals the volatility and consistency of returns, highlighting outliers and showing whether an industry's performance is concentrated or widely dispersed.

```python
importimport

```

### Step 3: Temporal Trend Analysis & Trajectory Tracking

Moving beyond static averages, I analyzed how profitability evolved over the years to identify growth cycles, volatility, and potential market downturns.

*   **Multi-level Aggregation (`groupby(['Year', 'comp_type'])`)**: I grouped the data by both fiscal year and industry sector. This granular aggregation was necessary to construct a time-series view, allowing me to track the specific performance trajectory of each sector annually rather than just a single snapshot.
*   **DataFrame Flattening (`.reset_index()`)**: After aggregation, the data existed in a multi-index format. I applied this method to convert the hierarchical index back into standard columns. This transformation was a technical prerequisite to ensure the visualization library (Seaborn) could correctly interpret the 'Year' and 'comp_type' dimensions for plotting.
*   **Time-Series Visualization (`sns.lineplot`)**: I generated a multi-line chart to visualize the historical trend of ROE. By utilizing the hue parameter, I distinctively color-coded each industry, enabling an immediate visual comparison of how different sectors reacted to economic shifts over the timeline.


```python
import
```

### Step 4: Granular Competitor Analysis & Leader Identification

Shifting focus from macro-trends to micro-level performance, I executed a granular analysis to identify the top-performing entities within each industry sector for the most recent fiscal year.

*   **Dynamic Sector Filtering & Boolean Indexing**: Instead of analyzing the market as a monolith, I drilled down into specific industries (e.g., Tech, FMCG, Real Estate) using Boolean masks. I isolated the data for the latest available year (`.max()`) to ensure the rankings reflected current market realities rather than historical averages.
*   **Performance Ranking Logic (`sort_values`)**: For every industry, I implemented a sorting algorithm based on Return on Equity (ROE) in descending order. By slicing the data with `.head(5)`, I effectively curated a "Best in Class" list, isolating market leaders from the long tail of average performers to pinpoint high-potential investment targets.
*   **Horizontal Bar Visualization (`sns.barplot` with `orient='h'`)**: I selected a horizontal orientation for the charts to visualize the top 5 performers. This was a deliberate design choice to accommodate lengthy company names on the Y-axis without overlapping, ensuring readability while facilitating a direct visual comparison of the top contenders' profitability margins.
  
```python
latest_year = merged_df['Year'].max()
all_industries = merged_df['comp_type'].unique()

for industry in all_industries:
    industry_df = merged_df[
        (merged_df['comp_type'] == industry) & 
        (merged_df['Year'] == latest_year)
    ]
    
    if industry_df.empty:
        continue

    top_5_performers = industry_df.sort_values(by='ROE', ascending=False).head(5)

    print(f"\n--- Top 5 Performers in '{industry}' Industry (Year: {latest_year}) ---")
    print(top_5_performers[['company', 'ROE', 'Net_Profit_Margin', 'Debt_to_Equity']].round(2))

    plt.figure(figsize=(12, 7))
    sns.barplot(data=top_5_performers, x='ROE', y='company', palette='viridis')
    plt.title(f'Top 5 Companies in "{industry}" Industry by ROE ({latest_year})', fontsize=16)
    plt.xlabel('Return on Equity (ROE)', fontsize=12)
    plt.ylabel('Company', fontsize=12)
    plt.tight_layout()
    plt.show()
```

---

## 4. Result

The analysis yielded clear, data-driven insights into the performance of the manufacturing line.

### Key Findings:

*   **High-Alert Operators**: **Operator Op-4** was identified as the most inconsistent, with an alert rate of **23.5%**. **Op-5 (20.7%)**, **Op-2 (19.1%)**, and **Op-16 (19.1%)** also showed significantly high rates of deviation.
*   **Top-Performing Operators**: Conversely, **Operator Op-18** was the most stable with an alert rate of only **4.0%**, followed by **Op-12 (6.3%)** and **Op-6 (7.1%)**.

### Data Visualization:
<img width="600" height="350" alt="{C120E15C-D9DD-472A-B2B6-5C9AF499C0B4}" src="https://github.com/user-attachments/assets/1774dd9c-e370-4de4-bc98-0a6121b4c755" />


### Actionable Recommendations & Business Impact:

*  **Prioritize Maintenance**: Immediately schedule inspections and recalibration for the machines handled by operators **Op-4, Op-5, and Op-2**. This targeted approach focuses resources where they are most needed.
*  **Standardize Best Practices**: Investigate the processes and techniques used by the top-performing operators **(Op-18, Op-12, Op-6)**. Their methods could be documented and used as a benchmark for training across the entire production floor.
