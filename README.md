# High-Quality Momentum Investing Strategy for S&P 500

This project implements a **Quantitative Momentum Strategy** in Python using the S&P 500 stock data. The strategy is built on identifying stocks with strong momentum by calculating the percentile rank for different time periods (1 month, 3 months, 6 months, 1 year) and selecting top-performing stocks. 
**NOTE: Backtest strategy currently not working, IGNORE.**

## Features

- Fetches S&P 500 tickers from Wikipedia.
- Downloads stock data for the past two years using `yfinance`.
- Calculates momentum percentiles for stocks over different time periods.
- Ranks stocks based on their momentum scores.
- Supports equal-weighted and momentum-weighted portfolio strategies.
- Outputs the strategy results to Excel files with formatting.
- Includes a simple backtest of the portfolio strategy against the S&P 500 benchmark.

## Library Imports

The strategy relies on the following Python libraries:

```python
import numpy as np
import pandas as pd
import requests
import math
from scipy import stats
import xlsxwriter
from datetime import datetime, timedelta
import yfinance as yf
from statistics import mean
```

# High-Quality Momentum (HQM) Strategy

## Strategy Overview
The **High-Quality Momentum (HQM) Strategy** focuses on selecting stocks based on their momentum over various time periods: 1-month, 3-month, 6-month, and 1-year. Stocks are ranked and scored according to their percentile ranks across these periods, allowing for a balanced approach that captures both short-term and long-term momentum.

## Steps Involved:

### 1. Fetching S&P 500 Tickers
We obtain the list of S&P 500 tickers from **Wikipedia** using the `pandas` library to dynamically fetch the latest list of companies in the S&P 500 index.

### 2. Date Handling
To analyze historical stock performance, we calculate:
- The **current date** 
- A date **two years prior**, which is used to gather stock data over this period.

### 3. Building the HQM Strategy
Momentum is measured by analyzing **price returns** over the following periods:
- 1 month
- 3 months
- 6 months
- 1 year

For each stock:
- **Percentiles** are calculated for each return period.
- The **HQM Score** is computed as the average of these percentiles.

### 4. Stock Selection
- The top **50 stocks** with the highest HQM scores are selected as part of the strategy.

## Portfolio Construction
The strategy can be implemented using two approaches:

1. **Equal-Weighted Strategy**:  
   Each stock is assigned an equal position size in the portfolio.

2. **Momentum-Weighted Strategy**:  
   Stocks with higher HQM scores receive larger weights in the portfolio, proportional to their momentum strength.

## Excel Output
The strategy results are exported to an Excel file with formatting using the `xlsxwriter` library. This includes price, number of shares to buy, momentum scores, and other metrics presented in a user-friendly format.

Example of Data Import
To fetch stock data and calculate returns:

```python

# Fetch stock data for past 2 years
data = yf.download(stock, start=two_years_before, end=current_date)

# Calculate returns for different time periods
return1yr = (close - closest_1yr) / closest_1yr if pd.notna(closest_1yr) else float('nan')
return6m = (close - closest_6m) / closest_6m if pd.notna(closest_6m) else float('nan')
return3m = (close - closest_3m) / closest_3m if pd.notna(closest_3m) else float('nan')
return1m = (close - closest_1m) / closest_1m if pd.notna(closest_1m) else float('nan')
```
# Momentum Percentile Calculation
For each stock, we calculate the percentile rank of its price return over the following periods:

```python
for row in hqm_dataframe.index:
    for time_period in time_periods:
        hqm_dataframe.loc[row, f'{time_period} Return Percentile'] = stats.percentileofscore(
            hqm_dataframe[f'{time_period} Price Return'], 
            hqm_dataframe.loc[row, f'{time_period} Price Return']
        )/100
```
# Calculating HQM Score
The **HQM Score** is the arithmetic mean of the momentum percentiles:

```python
for row in hqm_dataframe.index:
    momentum_percentiles = []
    for time_period in time_periods:
        momentum_percentiles.append(hqm_dataframe.loc[row, f'{time_period} Return Percentile'])
    hqm_dataframe.loc[row, 'HQM Score'] = mean(momentum_percentiles)
```

# Portfolio Allocation
For **equal weighting**, the number of shares to buy is determined as follows:

```python
position_size = float(portfolio_size) / len(hqm_dataframe.index)

for i in range(len(hqm_dataframe)):
    hqm_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / hqm_dataframe.loc[i, 'Price'])
```

For **momentum-weighted** strategy:
```python
total_momentum_score_sum = hqm_dataframe['HQM Score'].sum()
hqm_dataframe['Weight'] = hqm_dataframe['HQM Score'] / total_momentum_score_sum

for i in range(len(hqm_dataframe)):
    position_size = portfolio_size * hqm_dataframe.loc[i, 'Weight']
    hqm_dataframe.loc[i, 'Number of Shares to Buy'] = math.floor(position_size / hqm_dataframe.loc[i, 'Price'])
```

# Excel Output
The final results are saved to an Excel file with formatted columns for easy readability:

```python
writer = pd.ExcelWriter('momentum_strategy.xlsx', engine='xlsxwriter')
hqm_dataframe.to_excel(writer, sheet_name='Momentum Strategy', index=False)

# Apply formatting
for column in column_formats.keys():
    writer.sheets['Momentum Strategy'].set_column(f'{column}:{column}', 22, column_formats[column][1])

writer.close()

```
