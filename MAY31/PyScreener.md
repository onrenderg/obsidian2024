---

---

---
title: " 915Screener "
---

## Setup Env

```bash
echo -e "smartapi-python\npyotp\nlogzero\nstreamlit\nFlask\ndatetime\npandas\nnumpy" > requirements.txt

pip3 install -r re*
```


## Import and auth
```python

from SmartApi import SmartConnect #or from SmartApi.smartConnect import SmartConnect

from pyotp import TOTP

from logzero import logger

import pandas as pd

  
  
  
  

totp_key = 'T2PX34ABOGB3VZIVENMUAOJ5PQ'

api_key = 'kSJIOphE '

username = 'P547740'

pin = '8894'

obj = SmartConnect(api_key=api_key)

data = obj.generateSession(username, pin, TOTP(totp_key).now())
```
## GetNSETickList

```python
import requests as re

import pandas as pd

url = "https://margincalculator.angelbroking.com/OpenAPI_File/files/OpenAPIScripMaster.json"

d = re.get(url).json()

  

tk_df = pd.DataFrame.from_dict(d)

tk_df['expiry'] = pd.to_datetime(tk_df['expiry'])

dfx = tk_df.astype({'strike': float})

  
  

# Specify the unique value you want to search for

unique_value1 = 'NSE'

unique_value2 = 'BSE'

  

# Use boolean indexing to filter rows with the specified unique value

filtered_rows = dfx[(dfx['exch_seg'] == unique_value2) | (dfx['exch_seg'] == unique_value1) & dfx['symbol'].str.endswith('-EQ')]

filtered_rows = filtered_rows[filtered_rows['exch_seg'] == unique_value1]

# Print the filtered rows

NSElist = filtered_rows

  

dfxx = NSElist

## Check List
rows_with_bombaydyeing = NSElist[filtered_rows['name'] == 'BOMDYEING']

rows_with_bombaydyeing = rows_with_bombaydyeing.drop(rows_with_bombaydyeing.columns[1], axis=1)

rows_with_bombaydyeing

```

## DatetimeLogicForAutomaticDownload

```python
from datetime import datetime, date
import time
# Get today's date
today_date = date.today()

# Define the specific times
from_time = datetime(today_date.year, today_date.month, today_date.day, 9, 15)
to_time = datetime(today_date.year, today_date.month, today_date.day, 14, 30)

# Format the times
from_time_str = from_time.strftime('%Y-%m-%d %H:%M')
to_time_str = to_time.strftime('%Y-%m-%d %H:%M')


```

## Download rDataFrames
```python


def getCandleDatax(symbolInfo):

    try:
        historicParam={
        "exchange": symbolInfo.exch_seg,
        "symboltoken": symbolInfo.token,
        "interval": "ONE_MINUTE",
        "fromdate": from_time_str, #"2024-05-07 09:15" ,#from_time_str ,#f'{date.today()} 09:15' ,
        "todate": to_time_str  #"2024-05-07 14:30" # to_time_str  #f'{date.today()} 14:30'
        }
        res_json=  obj.getCandleData(historicParam)
        columns = ['timestamp','open','high','low','close','volume']
        df = pd.DataFrame(res_json['data'], columns=columns)
        # df['timestamp'] = pd.to_datetime(df['timestamp'],format = '%Y-%m-%dT%H:%M:%S')
        df['timestamp'] = pd.to_datetime(df['timestamp'], format='%Y-%m-%dT%H:%M:%S%z')
        df['symbol'] = symbolInfo.symbol
        print(f"Done for {symbolInfo.symbol}")
        time.sleep(0.4)
        return df
    except Exception as e:
        print(f"Historic Api failed: {e} {symbolInfo.symbol}")

#########################################################################


def fetch_candle_datax(eqSymbolDf=dfxx):
    eqdfList = []
    for i in eqSymbolDf.index:
        try:
            row_series = eqSymbolDf.iloc[i]
            candelRes = getCandleDatax(row_series)
            eqdfList.append(candelRes)
        except Exception as e:
            print(f"Fetching Historical Data failed for {row_series.symbol}: {e}")
    return eqdfList



```



## FlowStructure
```python

eqdfList = fetch_candle_datax()

eqdfList



```

## CheckDflist

```python
listx = []
dfn_count = 0

for dfn in eqdfList:
    if not dfn.empty:
        listx.append(dfn.iloc[-1]["symbol"])
        dfn_count += 1
    else:
        print("DataFrame is empty")

print("Length of listx:", len(listx))
print("Number of non-empty DataFrames (dfns):", dfn_count)
```
## ScreenerFunction

```python
def process_datax(eqdfList):
    listx = []
    for dfx in eqdfList:
        if not dfx.empty:  # Check if DataFrame is not empty
            last_row = dfx.iloc[-1]
            if last_row['close'] > last_row['open']: #2/3
                signal = 2
            elif last_row['close'] < last_row['open']: #1
                signal = 1
            else:
                signal = None

            # Append the signal to the DataFrame
            dfx['Signal'] = signal

            # Check if signal is generated
            if signal == 2:
                listx.append(last_row['symbol'])
            else:
                print("No signal generated for:", last_row['symbol'])
        else:
            print("DataFrame is empty")

    return listx


```

## Outpt

```
slist = process_datax(eqdfList)

  

slist
```


## Cron
```md
# tmux new -s app
# python app.py > log.txt 2>&1 &


# python-is-python3

# tmux new -s app
# python app.py > log.txt 2>&1 &
"""
cat > mycronjobs.txt << EOF
36 15 * * * python3 logix.py > log.txt 2>&1 & 
EOF
"""

# crontab -l 
# crontab -r  # Remove all existing cron jobs
# crontab mycronjobs.txt
# tail -f log.txt
# ps aux | grep "python3 app.py"
# ps aux | awk '!/grep/ && /python3 app.py/'
#sudo kill -9 68213
```