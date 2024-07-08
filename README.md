# Bovespa Dividend Yield Scanner

This project aims to scan and identify the stocks with the highest dividend yields on the Bovespa (São Paulo Stock Exchange) according to a user-specified reference year. The script uses the `yfinance` library to fetch historical closing prices and dividends of IBRX 50 index stocks for the past 5 years, calculates the annual Dividend Yield for the chosen reference year, and visualizes the results in a colorful bar chart.

## Features

1. **Data Download:**
   - Fetches closing prices and dividends of IBRX 50 index stocks using `yfinance`.

2. **Reference Year Filtering:**
   - Filters the fetched data for the specified reference year.

3. **Dividend Yield Calculation:**
   - Computes the average annual closing price and the total dividends paid in the specified year.
   - Calculates the annual Dividend Yield for each stock.

4. **Generation of Intermediate DataFrames:**
   - Creates intermediate DataFrames to store the average annual closing price and the total dividends paid, adding a column with the reference year.

5. **Data Plotting:**
   - Generates a colorful bar chart with stocks sorted by the highest Dividend Yield using `matplotlib`.

## Usage Instructions

1. **Dependencies Installation:**
   - Ensure you have `yfinance`, `pandas`, and `matplotlib` libraries installed. You can install them using `pip`:
     ```sh
     pip install yfinance pandas matplotlib
     ```

2. **Running the Script:**
   - Run the script to download data, calculate the Dividend Yield, and plot the chart. You can modify the reference year by changing the `year_reference` variable.

3. **Plotting Function:**
   - Use the `plot_dividend_yields(data, num_acoes=10)` function to plot the Dividend Yields of the stocks. The `num_acoes` parameter defines how many stocks will be plotted.

## Example

Here is an example of how to use the script and plotting function:

```python
import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
import random

# Specify the reference year
year_reference = 2023

# List of IBRX 50 tickers
tickets_ibrx50 = [
    'ABEV3.SA', 'ALPA4.SA', 'AMAR3.SA', 'ARZZ3.SA', 'ASAI3.SA', 'AZUL4.SA', 'B3SA3.SA', 
    'BBAS3.SA', 'BBDC3.SA', 'BBDC4.SA', 'BBSE3.SA', 'BEEF3.SA', 'BPAC11.SA', 'BRAP4.SA',
    'BRFS3.SA', 'BRKM5.SA', 'CCRO3.SA', 'CIEL3.SA', 'CMIG4.SA', 'COGN3.SA', 
    'CPFE3.SA', 'CRFB3.SA', 'CSAN3.SA', 'CSNA3.SA', 'CVCB3.SA', 'CYRE3.SA', 'ECOR3.SA', 'EGIE3.SA', 
    'ELET3.SA', 'ELET6.SA', 'EMBR3.SA', 'ENEV3.SA', 'ENGI11.SA', 'EQTL3.SA', 'EZTC3.SA', 
    'FLRY3.SA', 'GGBR4.SA', 'GOAU4.SA', 'GOLL4.SA', 'HAPV3.SA', 'HYPE3.SA',
    'IRBR3.SA', 'ITSA4.SA', 'ITUB4.SA', 'JBSS3.SA', 'KLBN11.SA', 'LREN3.SA', 'MGLU3.SA', 
    'MRFG3.SA', 'MRVE3.SA', 'MULT3.SA', 'NTCO3.SA', 'PCAR3.SA', 'PETR3.SA', 'PETR4.SA', 'PRIO3.SA', 
    'QUAL3.SA', 'RADL3.SA', 'RAIL3.SA', 'RENT3.SA', 'SBSP3.SA', 'SLCE3.SA', 'SUZB3.SA', 
    'TAEE11.SA', 'TOTS3.SA', 'UGPA3.SA', 'USIM5.SA', 'VALE3.SA', 'VIVT3.SA', 'WEGE3.SA', 
    'YDUQ3.SA'
]

# Create empty dataframes to store data
df_close = pd.DataFrame()
df_dividends = pd.DataFrame()

# Download closing prices and dividends for each individual ticker
for ticker in tickets_ibrx50:
    data = yf.Ticker(ticker)
    hist = data.history(period="5y")
    
    if 'Close' in hist.columns and 'Dividends' in hist.columns:
        # Filter only stocks that paid dividends in the last 5 years
        if hist['Dividends'].sum() > 0:
            df_close[ticker] = hist['Close']
            df_dividends[ticker] = hist['Dividends']

# Filter data for the reference year
df_close_year = df_close[df_close.index.year == year_reference]
df_dividends_year = df_dividends[df_dividends.index.year == year_reference]

# Calculate the average annual closing price and add the reference year column
avg_close_year = df_close_year.mean().to_frame(name='Average Close')
avg_close_year['Reference Year'] = year_reference

# Calculate the total dividends paid in the year and add the reference year column
total_dividends_year = df_dividends_year.sum().to_frame(name='Total Dividends')
total_dividends_year['Reference Year'] = year_reference

# Calculate the annual dividend yield and add the reference year column
dividend_yield_year = (total_dividends_year['Total Dividends'] / avg_close_year['Average Close']).sort_values(ascending=False).to_frame(name='Dividend Yield')
dividend_yield_year['Reference Year'] = year_reference

# Sort all stocks by highest to lowest dividend yield
ordered_dividend_yield = dividend_yield_year.sort_values(by='Dividend Yield', ascending=False)

# Store the dataframe with the necessary data
data = pd.DataFrame({
    'Ticker': ordered_dividend_yield.index,
    'Dividend Yield': ordered_dividend_yield['Dividend Yield'].values,
    'Reference Year': ordered_dividend_yield['Reference Year'].values
})

# Display the final dataframe
print(data)
print(avg_close_year)
print(total_dividends_year)

# Function to plot the data
def plot_dividend_yields(data, num_acoes=10):
    """
    Plots the Dividend Yields of the stocks in a bar chart.

    Parameters:
    - data: DataFrame containing the columns 'Ticker', 'Dividend Yield', and 'Reference Year'
    - num_acoes: Number of stocks to be plotted (default is 10)
    """
    # Sort data by Dividend Yield
    data_sorted = data.sort_values(by='Dividend Yield', ascending=False).head(num_acoes)

    # Generate random colors
    colors = ['#%06X' % random.randint(0, 0xFFFFFF) for _ in range(num_acoes)]

    # Plot the bar chart
    plt.figure(figsize=(12, 8))
    plt.bar(data_sorted['Ticker'], data_sorted['Dividend Yield'], color=colors)
    plt.xlabel('Ticker')
    plt.ylabel('Dividend Yield')
    plt.title(f'Top {num_acoes} Stocks by Dividend Yield - {data_sorted["Reference Year"].iloc[0]}')
    plt.xticks(rotation=45)
    plt.tight_layout()
    plt.show()

# Example of using the function
plot_dividend_yields(data, num_acoes=10)

# Contributing
If you find any issues or have suggestions for improvements, please create an issue or submit a pull request.

# License
This project is licensed under the MIT License.




