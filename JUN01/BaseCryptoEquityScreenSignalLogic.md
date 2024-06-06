---
title: "Botv1"
---

## mainBlocks

### SignalScreenrBlock

#### Crypto

```python
import pandas as pd
import pprint

def check_conditionsx(df, numx=2, num_candles=3):
    if len(df) < numx:
        return None
    
    macd_condition = False
    for i in range(1, numx):
        if df.iloc[-i - 1]['macd_linex'] < df.iloc[-i - 1]['macd_signalx']:
            macd_condition = True
            print(f"MACD condition satisfied at index {-i}")
            break  # Once the condition is satisfied, no need to check further

    candle_condition = True  # Initialize as True
    for i in range(1, num_candles + 1):
        if df.iloc[-i - 1]['close'] < df.iloc[-i - 1]['open']:
            candle_condition = False
            print(f"Candle condition not satisfied at index {-i}")
            break  # Once a candle doesn't meet the condition, no need to check further

    print("MACD condition:", macd_condition)
    print("Candle condition:", candle_condition)

    if macd_condition and candle_condition and df.iloc[-1]['close'] > df.iloc[-1]['open']:
        return df.iloc[-1]['time']  # Return the time of the last candle if conditions are met
    else:
        return False


df = pd.read_csv("macd.csv")
df.drop(df.columns[0], axis=1, inplace=True)
df['macd_linex'] = df.macd_signal
df['macd_signalx' ]= df.macd_line

# Convert date column to datetime
df['time'] = pd.to_datetime(df['time'])

# Define start and end dates
start_date = pd.to_datetime('2024-05-11 14:54:00')
end_date = pd.to_datetime('2024-05-12 05:18:00')

# Slice the DataFrame based on the date range
sliced_df = df[(df['time'] >= start_date) & (df['time'] <= end_date)]

# Reset index and drop the old index column
sliced_df.reset_index(drop=True, inplace=True)

# Slice the sliced_df to get the first 7 rows
rows = sliced_df.iloc[:7]

# Initialize an empty DataFrame for concatenation
dfx = pd.DataFrame(rows)

# Concatenate the rows to dfx
for i in range(8, len(sliced_df)):
    row_data = sliced_df.iloc[i]
    row_data_df = pd.DataFrame([row_data], index=[i])
    dfx = pd.concat([dfx, row_data_df])
    result = check_conditionsx(dfx)
    if result:
        print("Time of the last candle meeting conditions:", result)



```

### Stock
```python
import pandas as pd
import pprint

def check_conditionsx(df, numx=2, num_candles=3, rsi_threshold=29.31):
    if len(df) < numx:
        return None
    
    macd_condition = False
    for i in range(1, numx):
        if df.iloc[-i - 1]['macd_linex'] < df.iloc[-i - 1]['macd_signalx']:
            macd_condition = True
            print(f"MACD condition satisfied at index {-i}")
            break  # Once the condition is satisfied, no need to check further

    candle_condition = True  # Initialize as True
    for i in range(1, num_candles + 1):
        if df.iloc[-i - 1]['close'] < df.iloc[-i - 1]['open']:
            candle_condition = False
            print(f"Candle condition not satisfied at index {-i}")
            break  # Once a candle doesn't meet the condition, no need to check further

    
    rsi_condition = df.iloc[-1]['rsi'] < rsi_threshold
    
    
    print("MACD condition:", macd_condition)
    print("Candle condition:", candle_condition)

    
    if macd_condition and candle_condition and df.iloc[-1]['close'] > df.iloc[-1]['open']  and rsi_condition:
        return df.iloc[-1]['time']  # Return the time of the last candle if conditions are met
    else:
        return False


df = pd.read_csv("macd.csv")
df.drop(df.columns[0], axis=1, inplace=True)
df['macd_linex'] = df.macd_signal
df['macd_signalx' ]= df.macd_line

# Convert date column to datetime
df['time'] = pd.to_datetime(df['time'])

# Define start and end dates
start_date = pd.to_datetime('2024-05-11 14:54:00')
end_date = pd.to_datetime('2024-05-12 05:18:00')

# Slice the DataFrame based on the date range
sliced_df = df[(df['time'] >= start_date) & (df['time'] <= end_date)]

# Reset index and drop the old index column
sliced_df.reset_index(drop=True, inplace=True)

# Slice the sliced_df to get the first 7 rows
rows = sliced_df.iloc[:7]

# Initialize an empty DataFrame for concatenation
dfx = pd.DataFrame(rows)

# Concatenate the rows to dfx
for i in range(8, len(sliced_df)):
    row_data = sliced_df.iloc[i]
    row_data_df = pd.DataFrame([row_data], index=[i])
    dfx = pd.concat([dfx, row_data_df])
    result = check_conditionsx(dfx)
    if result:
        print("Time of the last candle meeting conditions:", result)


```