


```txt
smartapi-python

pyotp

logzero

streamlit

Flask

datetime

pandas

numpy
```


## Gen DateList & StrikePriceList

```python
# generate python function to create a list of number starting from x1 like 17350 to x2 like 21300 and each element of list should be 50 points number difference so [17350, 17400, 17450, 17500 ..... 21250,21300]


def generate_number_list(x1, x2, step=50):
    number_list = []
    current_number = x1

    while current_number <= x2:
        number_list.append(current_number)
        current_number += step

    return number_list

# Example usage:
x1 = 21800 #small
x2 = 22200 #large
result = generate_number_list(x1, x2)
print(result)

```
```bash
[21800, 21850, 21900, 21950, 22000, 22050, 22100, 22150, 22200]
```



```python

import pandas as pd
import requests as re
import time as tt
import pyotp
from SmartApi import SmartConnect
import os
import zipfile
import shutil

```



```python
expiri = "2024-05-09" #mongo1
date_list = ['2024-03-26']  # Add more dates as needed
strike_prices = [21800, 21850, 21900, 21950, 22000, 22050, 22100, 22150, 22200]
intervals = ["ONE_MINUTE", "THREE_MINUTE", "FIFTEEN_MINUTE", "THIRTY_MINUTE", "ONE_HOUR"]



def init_symbol_token_map():
    url = "https://margincalculator.angelbroking.com/OpenAPI_File/files/OpenAPIScripMaster.json"
    data = re.get(url).json()
    tk_df = pd.DataFrame.from_dict(data)
    tk_df['expiry'] = pd.to_datetime(tk_df['expiry'])
    tk_df = tk_df.astype({'strike': float})
    return tk_df




def get_symbol_info(exch_seg, instrumenttype, symbol, strike_price, option_type):
    df = init_symbol_token_map()
    strike_price = strike_price * 100
    if exch_seg == 'NFO' and (instrumenttype == 'OPTIDX'):
        return df[(df['exch_seg'] == exch_seg) & (df['instrumenttype'] == instrumenttype) 
        & (df['name'] == symbol) & (df['strike'] == strike_price)                      
        & (df['symbol'].str.endswith(option_type) & (df['expiry'] == expiri))].sort_values(by='symbol')

```



```python
def get_candle_data(symbolInfo, interval, fromdate, todate):

    historic_param = {
    "exchange": symbolInfo.exch_seg,
    "symboltoken": symbolInfo.token,
    "interval": interval,
    "fromdate": fromdate,
    "todate": todate
}

    try:

        
        res_json = obj.getCandleData(historic_param)
        columns = ['timestamp', 'open', 'high', 'low', 'close', 'volume']
        df = pd.DataFrame(res_json['data'], columns=columns)
        df['timestamp'] = pd.to_datetime(df['timestamp'], format='%Y-%m-%dT%H:%M:%S')
        df['symbol'] = symbolInfo.symbol
        df['expiry'] = symbolInfo.expiry
        print(f"Done for {symbolInfo.symbol} - {interval}")
        tt.sleep(0.2)
        return df
    except Exception as e:
        print(f"Historic API failed: {e} {symbolInfo.symbol} - {interval}")

```


```python

# Function to move zip files to Google Drive
def move_zip_file_to_drive(date):
    # zip_file_name = f"data_csv_{date}.zip"
    # source_path = zip_file_name
    # destination_path = f'/content/drive/MyDrive/Options/NIFTY_sessions/24-04-26/{zip_file_name}'
    # !mv $source_path $destination_path
    # print(f"Moved {zip_file_name} to Google Drive")
    print(date)
```



## Flow

* After gen date-list :

```python
for date in date_list:
    shutil.rmtree(save_directory, ignore_errors=True)  # Remove existing contents
    os.makedirs(save_directory)  # Recreate the directory

    zip_file_name = f"data_csv_{date}.zip"
    with zipfile.ZipFile(zip_file_name, 'w', zipfile.ZIP_DEFLATED) as zipf:
        for strike_price in strike_prices:
            for option_type in ["CE", "PE"]:
                symbol_info = get_symbol_info('NFO', 'OPTIDX', 'NIFTY', strike_price, option_type).iloc[0] #mongo2
                # Create directory for this strike price and option type
                directory_name = f"{symbol_info.symbol}_{strike_price}_{option_type}"
                directory_path = os.path.join(save_directory, directory_name)
                os.makedirs(directory_path, exist_ok=True)
                for interval in intervals:
                    fromdate = f'{date} 09:15'
                    todate = f'{date} 15:30'
                    data_frame = get_candle_data(symbol_info, interval, fromdate, todate)

                    file_name = f"{symbol_info.symbol}_{strike_price}_{interval}_{option_type}.csv"
                    file_path = os.path.join(directory_path, file_name)
                    data_frame.to_csv(file_path, index=False)

                    # Iterate through files in the directory
                    for filename in os.listdir(directory_path):
                        file_path = os.path.join(directory_path, filename)

                        # Add file to zip with directory structure
                        zipf.write(file_path, os.path.join(directory_name, filename))

    # Move the zip file to Google Drive
    move_zip_file_to_drive(date)
```

[[final_impt_kjsdlf]]
[[refactor-lksddfl]]