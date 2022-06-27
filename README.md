
# CRYPTOCURRENCY DATA SCRAPING WITH PYTHON

In this project, I used Python to scrape cryptocurrency data from the Coindesk API and also performed exploratory data analysis to learn more about the data.


## Libraries/Packages used - Numpy, Pandas, requests, datetime.
## Usage/Examples

```javascript
# Importing necessary Libraries

import requests
import pandas as pd
import numpy as np
from datetime import datetime
from dateutil.relativedelta import relativedelta

# The list of Top 20 CryptoCurenncy

coin_list = ['BTC', 'ETH', 'XRP', 'ADA', 'USDT', 'DOGE', 'XLM', 'DOT', 'UNI', 'LINK', 'USDC', 'BCH', 'LTC', 'GRT', 'ETC', 
             'FIL', 'AAVE', 'ALGO', 'EOS']

# Creating DataFrame

main_df = pd.DataFrame()

# Code for scraping the Data

for coin in coin_list:
    coin_df = pd.DataFrame()
    df = pd.DataFrame(index=[0])
    
    # Defining the Start Date and End Date
    datetime_end = datetime(2021, 7, 2, 0, 0)
    datetime_check = datetime(2021, 7, 1, 0, 0)
    
    while len(df) > 0:
        if datetime_end == datetime_check:
            break
        
        datetime_start = datetime_end - relativedelta(hours = 12)
        
        #Api for the scrapping
        url = 'https://production.api.coindesk.com/v2/price/values/'+ coin +'?start_date='+datetime_start.strftime("%Y-%m-%dT%H:%M") + '&end_date=' + datetime_end.strftime("%Y-%m-%dT%H:%M") + '&ohlc=true'
        
        #we are using the request to fetch the data from the api in the json format and then storing it into the dataframe.
        temp_data = requests.get(url).json()
        df = pd.DataFrame(temp_data['data']['entries'])
        df.columns = ['Timestamp', 'Open', 'High', 'Low', 'Close']
        
        # To handle the Missing Data
        insert_ids_list = [np.nan]
        
        
        
        while len(insert_ids_list) > 0:
            timestamp_checking = np.array(df['Timestamp'][1:]) - np.array(df['Timestamp'][:-1])
            insert_ids_list = np.where(timestamp_checking!= 60000)[0]
            if len(insert_ids_list) > 0:
                print(str(len(insert_ids_list)) + ' mismatched.')
                insert_ids = insert_ids_list[0]
                temp_df = df.iloc[insert_ids.repeat(int(timestamp_checking[insert_ids]/60000)-1)].reset_index(drop=True)
                temp_df['Timestamp'] = [temp_df['Timestamp'][0] + i*60000 for i in range(1, len(temp_df)+1)]
                df = df.loc[:insert_ids].append(temp_df).append(df.loc[insert_ids+1:]).reset_index(drop=True)
                insert_ids_list = insert_ids_list[1:]
                
        
        #adding datetime and symbol to dataframe
        df = df.drop(['Timestamp'], axis=1)
        df['Datetime'] = [datetime_end - relativedelta(minutes=len(df)-i) for i in range(0, len(df))]
        coin_df = df.append(coin_df)
        datetime_end = datetime_start
        
    coin_df['Symbol'] = coin
    main_df = main_df.append(coin_df)

    main_df = main_df[['Datetime', 'Symbol', 'Open', 'High', 'Low', 'Close']].reset_index(drop=True)
    main_df

    # Saving The Scrapped data into csv file

    main_df.to_csv('main_df.csv', index=False)
   
```
Observation/Insights after performing EDA

1 - Bitcoin is the highest value according to market cap and etherum is 50% of it and all other are very less in comperision to it and USDT is 3 times less then the Etherum
2 - Investment in penny cryptocurrencies should be avoided, as depicted by the candlestick chart of USDT.
3 - In 2021 the value of BTC was at all-time high of nearly 60,000$ which is almost 15 times more than the second-highest cryptocurrency ETH .
4 - Among the top cryptocurrencies, the growth of BTC, ETH, and USTD over the last five years was beneficial for the investors.
