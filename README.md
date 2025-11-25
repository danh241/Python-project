# Cross-Industry Financial Performance Analysis with Python

## Project Summary

This project demonstrates the application of Python for comprehensive financial statement analysis to evaluate corporate performance. By integrating raw Balance Sheet and Income Statement data, I engineered key financial ratios (ROE, ROA, Net Profit Margin) to conduct a multi-faceted analysis. The project moves from descriptive industry benchmarking and temporal trend tracking to a diagnostic assessment of the risk-return trade-off. The final analysis provides data-driven insights into how capital structure decisions (Leverage) impact shareholder value across different sectors.

**Tools Used:** Python (Pandas for Data Manipulation, NumPy, Seaborn & Matplotlib for Data Visualization)

**Skills Demonstrated:** Data Cleaning & Transformation, Feature Engineering (Financial Ratios), Exploratory Data Analysis (EDA), Trend Analysis, Correlation & Statistical Diagnosis

---

## 1. Situation

An investment research team was facing challenges in efficiently evaluating the performance of a diverse portfolio of companies across the Technology, FMCG, and Real Estate sectors. Critical financial data was siloed in disparate Excel files (Balance Sheets and Income Statements), making manual cross-industry comparison time-consuming and prone to human error. Investment decisions were often made based on static snapshots rather than dynamic, historical trends, leading to a lack of clarity regarding the risk-return trade-off. The primary business goal was to develop a scalable, automated Python pipeline to consolidate raw financial data, calculate advanced performance metrics, and visualize key insights to support data-driven capital allocation strategies.

---

## 2. Task

As a Data Analyst, my objective was to engineer a robust analytical framework to evaluate the financial health and operational efficiency of companies across multiple sectors using Python. My key responsibilities include:

*   **Data Integration & Cleaning**: Consolidate disparate financial datasets (Balance Sheet & Income Statement) into a unified structure, ensuring data integrity by systematically handling missing values and removing duplicates.
*   **Financial Ratio Engineering**: Programmatically calculate key performance indicators (KPIs) such as Return on Equity (ROE) and Debt-to-Equity to translate raw accounting figures into comparable metrics of profitability and leverage.
*   **Multidimensional Performance Visualization**: Develop a suite of visualizations to benchmark industry performance, track historical trends, and diagnose the structural relationship between financial leverage (risk) and shareholder returns.

---

## 3. Action

My analytical process was executed entirely within , focusing on efficiency and readability.

### Step 1: Data Consolidation & Financial Metric Engineering

I began by establishing a unified data foundation using Pandas to merge disparate financial statements and compute key performance indicators (KPIs) for company analysis.

*   **`pd.merge` with inner join**: This was essential to combine the Balance Sheet and Income Statement datasets. By joining on specific keys (`company`, `comp_type`, `Year`), I ensured that every financial ratio was calculated based on perfectly aligned data points for each specific fiscal year and entity.
*   **Handling Division by Zero (`.replace(0, np.nan)`)**: In financial data, denominators like 'Total Revenue' or 'Total Liabilities' can occasionally be zero. I proactively replaced these zeros with NaN (Not a Number) within the calculation formulas. This robust approach prevents program crashes due to mathematical errors and flags data gaps for transparency.
*   **`Calculated Attributes (Feature Engineering)`**: I programmed the logic for critical financial ratios such as ROE, ROA, and Debt-to-Equity. Specifically, I defined Operating Income as a proxy for net income to isolate and analyze the core operational efficiency of the companies, filtering out non-operational noise.

```python
# --- DATA CONSOLIDATION ---

import numpy as np
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt

# 1. Load Datasets
df_balance_sheet = pd.read_excel("data/Balance_Sheet.xlsx")
df_income_statement = pd.read_excel("data/Income_Statement.xlsx")

# 2. Merge Datasets
# Combining Balance Sheet and Income Statement based on Company, Type, and Year
merged_df = pd.merge(
    df_balance_sheet,
    df_income_statement,
    on=['company', 'comp_type', 'Year'],
    how='inner'
)

# 3. Data Quality Checks
print(merged_df.isnull().sum())
num_duplicates = merged_df.duplicated().sum()
print(f"Total Duplicates: {num_duplicates}")
merged_df.info()

# --- FINANCIAL METRIC ENGINEERING ---

# Define the profit metric used for calculations
net_income_proxy = 'Operating Income'

# Calculate Key Financial Ratios
# Using .replace(0, np.nan) to handle potential division by zero errors

# 1. Profitability Ratios
merged_df['Net_Profit_Margin'] = merged_df[net_income_proxy] / merged_df['Total Revenue'].replace(0, np.nan)
merged_df['ROE'] = merged_df[net_income_proxy] / merged_df['Total Stockholder Equity'].replace(0, np.nan)
merged_df['ROA'] = merged_df[net_income_proxy] / merged_df['Total Assets'].replace(0, np.nan)

# 2. Leverage & Liquidity Ratios
merged_df['Debt_to_Equity'] = merged_df['Total Liab'] / merged_df['Total Stockholder Equity'].replace(0, np.nan)
merged_df['Current_Ratio'] = merged_df['Total Current Assets'] / merged_df['Total Current Liabilities'].replace(0, np.nan)

# Create a final clean view with selected columns
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

print(final_view.round(2).head(10))
```

### Step 2: Comparative Industry Analysis & Visualization

After computing the individual ratios, I proceeded to aggregate the data to derive industry-level insights, utilizing Python's visualization libraries to uncover patterns in profitability and risk distribution.

*   **`groupby()` aggregation**: I segmented the dataset by industry sector (`comp_type`) to calculate the mean values for key metrics like ROE and Debt-to-Equity. This pivots the analysis from granular company details to broader sectoral benchmarks, establishing a baseline for performance.
*   **`sns.barplot` for Comparison**: I utilized a Seaborn bar chart to visualize the average Return on Equity (ROE) across different industries. This visualization provides an immediate, high-level ranking of which sectors are delivering the highest returns to shareholders on average.
*   **`sns.boxplot` for Distribution Analysis**: Recognizing that averages can be misleading, I implemented a box plot to examine the spread of ROE within each industry. This reveals the volatility and consistency of returns, highlighting outliers and showing whether an industry's performance is concentrated or widely dispersed.

```python
# --- ANALYSIS 1: INDUSTRY COMPARISON ---

industry_performance = merged_df.groupby('comp_type')[['Net_Profit_Margin', 'ROE', 'Debt_to_Equity']].mean()

print("\n Average Performance by Industry")
print(industry_performance.round(2))

# 1. Bar Chart: Average ROE by Industry
plt.figure(figsize=(10, 6))
sns.barplot(x=industry_performance.index, y='ROE', data=industry_performance)
plt.title('Average Return on Equity (ROE) by Industry', fontsize=16)
plt.xlabel('Industry Sector', fontsize=12)
plt.ylabel('Average ROE', fontsize=12)
plt.show()

# 2. Box Plot: ROE Distribution by Industry
plt.figure(figsize=(10, 6))
sns.boxplot(x='comp_type', y='ROE', data=merged_df)
plt.title('Distribution of ROE Across Industries', fontsize=16)
plt.xlabel('Industry Sector', fontsize=12)
plt.ylabel('Return on Equity (ROE)', fontsize=12)
plt.show()

```

### Step 3: Temporal Trend Analysis & Trajectory Tracking

Moving beyond static averages, I analyzed how profitability evolved over the years to identify growth cycles, volatility, and potential market downturns.

*   **Multi-level Aggregation (`groupby(['Year', 'comp_type'])`)**: I grouped the data by both fiscal year and industry sector. This granular aggregation was necessary to construct a time-series view, allowing me to track the specific performance trajectory of each sector annually rather than just a single snapshot.
*   **DataFrame Flattening (`.reset_index()`)**: After aggregation, the data existed in a multi-index format. I applied this method to convert the hierarchical index back into standard columns. This transformation was a technical prerequisite to ensure the visualization library (Seaborn) could correctly interpret the 'Year' and 'comp_type' dimensions for plotting.
*   **Time-Series Visualization (`sns.lineplot`)**: I generated a multi-line chart to visualize the historical trend of ROE. By utilizing the hue parameter, I distinctively color-coded each industry, enabling an immediate visual comparison of how different sectors reacted to economic shifts over the timeline.


```python
# --- ANALYSIS 2: TREND ANALYSIS OVER TIME ---

trend_analysis = merged_df.groupby(['Year', 'comp_type'])['ROE'].mean().reset_index()

print("\n Average ROE Trends Over Years")
print(trend_analysis.round(2))

# Line Chart: ROE Trends
plt.figure(figsize=(12, 7))
sns.lineplot(data=trend_analysis, x='Year', y='ROE', hue='comp_type', marker='o', linewidth=2.5)
plt.title('Average ROE Trends Over Time by Industry', fontsize=16)
plt.xlabel('Year', fontsize=12)
plt.ylabel('Average ROE', fontsize=12)
plt.legend(title='Industry Sector')
plt.xticks(trend_analysis['Year'].unique()) 
plt.show()
```

### Step 4: Granular Competitor Analysis & Leader Identification

Building on the descriptive and trend analyses, I advanced to a diagnostic phase to examine the interdependencies between financial metrics. I aimed to test the hypothesis of whether higher financial leverage (risk) correlates with superior shareholder returns (performance).

*   **`df.corr()` & Heatmap**: I computed the pairwise correlation coefficients for all numerical variables to identify strong positive or negative relationships. Visualizing this matrix with a Seaborn heatmap provided a "bird's-eye view" of the data, instantly highlighting key drivers of performance and ensuring no multicollinearity issues existed before further analysis.
*   **`sns.scatterplot` for Multivariate Analysis**: To investigate the structural relationship between risk and return, I implemented a scatter plot contrasting Debt-to-Equity against ROE by mapping Industry to hue and Net Profit Margin to point size, this visualization adds depth (3rd and 4th dimensions) to the analysis. This reveals whether companies taking on high leverage are successfully converting that debt into efficient returns, or if they are merely exposing themselves to higher risk without a proportional reward.
  
```python
# --- ANALYSIS 3: CORRELATION & RISK-RETURN DIAGNOSTIC ---

corr_matrix = merged_df[['Net_Profit_Margin', 'ROE', 'ROA', 'Debt_to_Equity', 'Current_Ratio']].corr()

print("\n Correlation Matrix")
print(corr_matrix.round(2))

# 1. Heatmap: Financial Metrics Correlation
plt.figure(figsize=(10, 8))
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm', fmt=".2f", linewidths=0.5)
plt.title('Correlation Heatmap of Financial Metrics', fontsize=16)
plt.show()

# 2. Scatter Plot: Risk vs. Return
plt.figure(figsize=(12, 7))
sns.scatterplot(data=merged_df, 
                x='Debt_to_Equity', 
                y='ROE', 
                hue='comp_type', 
                size='Net_Profit_Margin',
                sizes=(50, 400), 
                alpha=0.7)

plt.title('Risk-Return Analysis: Leverage (Debt/Equity) vs. Performance (ROE)', fontsize=16)
plt.xlabel('Debt to Equity Ratio (Leverage)', fontsize=12)
plt.ylabel('Return on Equity (ROE)', fontsize=12)
plt.legend(bbox_to_anchor=(1.05, 1), loc='upper left', title='Industry & Margin')
plt.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout()
plt.show()
```

---

## 4. Result

The comprehensive analysis of the dataset revealed distinct financial profiles for each industry, highlighting significant disparities in capital efficiency and growth momentum.

### Key Findings:

*   **Technology Dominates in Growth & Efficiency, FMCG Offers Stability**: The Technology sector achieves the highest average Return on Equity (ROE ~53%), tied with FMCG. However, the Box Plot reveals a critical nuance: Tech's performance is driven by high-growth outliers (likely successful startups or tech giants), whereas FMCG displays a much tighter distribution.

      <img width="436" height="291" alt="{7B4D41DB-90A9-4CB6-9028-00FF6A72ECC7}" src="https://github.com/user-attachments/assets/80e4f039-fa45-4316-a45d-3d58e88f449b" />
        <img width="436" height="291" alt="{C67BAE82-D3F6-48F1-ABBB-AF5566077210}" src="https://github.com/user-attachments/assets/e90aedee-2318-4212-a5c7-f01e17f0adf2" />


*   **Momentum Shift – The Post-2020 Tech Boom**: Temporal analysis exposes a dramatic divergence starting in 2020. While Real Estate remained stagnant and FMCG saw a gradual decline (from 0.60 to 0.41), the Technology sector experienced an exponential surge in ROE, peaking at 1.43 in 2022.

     <img width="510" height="322" alt="{6286635B-DAFB-4CB3-AF74-1A6931454181}" src="https://github.com/user-attachments/assets/cbdf9901-dcf9-4522-b58b-85d3cd14d821" />


*   **The Leverage Trap – High Debt Does Not Guarantee High Returns**: This is the most critical diagnostic finding. The Scatter Plot shows two distinct clusters:
Real Estate (Green Bubbles): High Financial Leverage (Debt-to-Equity > 6.0) but moderate ROE, Technology (Purple Bubbles): Low Financial Leverage (Debt-to-Equity < 2.0) but superior ROE.

      <img width="599" height="349" alt="{A452967E-B1AE-4EA1-83BB-2D98608A711C}" src="https://github.com/user-attachments/assets/8046492c-d7d3-4c6e-a7a0-c00d57e05c10" />


### Actionable Recommendations & Business Impact:

*  **Strategic Capital Allocation**: Shift portfolio weight towards the Technology sector for growth-focused funds, as they demonstrate the ability to scale returns without leveraging the balance sheet. Maintain FMCG holdings only as a defensive hedge against market volatility, but monitor the declining ROE trend closely.
*  **Risk Mitigation**: The Real Estate sector is showing a "High Risk (Debt), Moderate Return" profile. Recommendation to limit exposure to this sector in a rising interest rate environment, as their high debt burdens could severely erode future net profit margins.
*  **Operational Focus**:  For FMCG companies, investigate the root cause of the steady ROE decline since 2018. Is it pricing power, supply chain costs, or marketing inefficiency? A deeper dive into Operating Expenses is recommended
