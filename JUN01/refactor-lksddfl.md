
```python
import os
import shutil
import zipfile
import pandas as pd
import requests
import time
import pyotp

def generate_number_list(x1, x2, step=50):
    number_list = []
    current_number = x1

    while current_number <= x2:
        number_list.append(current_number)
        current_number += step

    return number_list

def init_symbol_token_map():
    url = "https://margincalculator.angelbroking.com/OpenAPI_File/files/OpenAPIScripMaster.json"
    data = requests.get(url).json()
    tk_df = pd.DataFrame.from_dict(data)
    tk_df['expiry'] = pd.to_datetime(tk_df['expiry'])
    tk_df = tk_df.astype({'strike': float})
    return tk_df

def get_symbol_info(exch_seg, instrumenttype, symbol, strike_price, option_type, expiri):
    df = init_symbol_token_map()
    strike_price = strike_price * 100
    if exch_seg == 'NFO' and (instrumenttype == 'OPTIDX'):
        return df[(df['exch_seg'] == exch_seg) & (df['instrumenttype'] == instrumenttype) 
                  & (df['name'] == symbol) & (df['strike'] == strike_price) 
                  & (df['symbol'].str.endswith(option_type) & (df['expiry'] == expiri))].sort_values(by='symbol')

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
        time.sleep(0.2)
        return df
    except Exception as e:
        print(f"Historic API failed: {e} {symbolInfo.symbol} - {interval}")

def move_zip_file_to_drive(date):
    # Implement the logic to move the zip file to Google Drive
    print(date)

def process_date(date, save_directory, strike_prices, intervals):
    shutil.rmtree(save_directory, ignore_errors=True)
    os.makedirs(save_directory)

    zip_file_name = f"data_csv_{date}.zip"
    with zipfile.ZipFile(zip_file_name, 'w', zipfile.ZIP_DEFLATED) as zipf:
        for strike_price in strike_prices:
            for option_type in ["CE", "PE"]:
                symbol_info = get_symbol_info('NFO', 'OPTIDX', 'NIFTY', strike_price, option_type, expiri).iloc[0]
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

                    for filename in os.listdir(directory_path):
                        file_path = os.path.join(directory_path, filename)
                        zipf.write(file_path, os.path.join(directory_name, filename))

    move_zip_file_to_drive(date)

def main(date_list, save_directory, strike_prices, intervals):
    for date in date_list:
        process_date(date, save_directory, strike_prices, intervals)

# Assuming expiri, date_list, save_directory, strike_prices, and intervals are defined
main(date_list, save_directory, strike_prices, intervals)

```

