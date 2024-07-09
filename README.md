# ExchangeRates_to_SQLite_with_Timestamp

This Python script scrapes currency exchange rates from the Bank of Taiwan's website, adds a timestamp, and stores the data in an SQLite database.

## Sample Code

```python
import requests
from bs4 import BeautifulSoup
import pandas as pd
import sqlite3
from datetime import datetime

# Define the URL to scrape
url = 'https://rate.bot.com.tw/xrt?Lang=zh-TW'

# Send a GET request to the URL
r = requests.get(url)

# Parse the HTML content using BeautifulSoup with html5lib parser
soup = BeautifulSoup(r.text, 'html5lib')

# Find all the currency names
currency = soup.find_all('div', class_="visible-phone print_hide")

# Find all the cash buy rates
cash_buy_rate = soup.find_all('td', class_="rate-content-cash text-right print_hide", attrs={'data-table': '本行現金買入'})

# Find all the cash sell rates
cash_sell_rate = soup.find_all('td', class_='rate-content-cash text-right print_hide', attrs={'data-table': '本行現金賣出'})

# Find all the sight buy rates
sight_buy_rate = soup.find_all('td', class_='rate-content-sight text-right print_hide', attrs={'data-table': '本行即期買入'})

# Find all the sight sell rates
sight_sell_rate = soup.find_all('td', class_='rate-content-sight text-right print_hide', attrs={'data-table': '本行即期賣出'})

# Initialize an empty list to store the data
data_list = []

# Get the current timestamp
timestamp = datetime.now().strftime('%y-%m-%d %H:%M:%S')

for i in range(len(currency)):
    # Strip any whitespace characters in currency_names, cash_buy, cash_sell, sight_buy, and sight_sell
    currency_name = currency[i].text.strip()
    cash_buy = cash_buy_rate[i].text.strip()
    cash_sell = cash_sell_rate[i].text.strip()
    sight_buy = sight_buy_rate[i].text.strip()
    sight_sell = sight_sell_rate[i].text.strip()

    # Append the extracted data to the list along with the timestamp
    data_list.append([currency_name, cash_buy, cash_sell, sight_buy, sight_sell, timestamp])

# Create a DataFrame from the list with the specified column names
df = pd.DataFrame(data_list, columns=['幣別','現金買入','現金賣出','即期買入','即期賣出','更新時間'])

# Connect to the SQLite database 
conn = sqlite3.connect('exchange_rates.db')

# Create a cursor object
c = conn.cursor()

# Create the exchange_rates table if it doesn't exist
c.execute(
    '''
    CREATE TABLE IF NOT EXISTS exchange_rates(
        id INTEGER PRIMARY KEY AUTOINCREMENT,
         幣別 TEXT,
         現金買入 TEXT,
         現金賣出 TEXT,
         即期買入 TEXT,
         即期賣出 TEXT,
         更新時間 TEXT
    )
    '''
)

# Insert the data into the exchange_rates table
df.to_sql('exchange_rate', conn, if_exists='append', index=False)
# Commit the transaction and close the connection
conn.commit()
conn.close()

print(df)

