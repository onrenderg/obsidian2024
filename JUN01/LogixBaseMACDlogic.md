```js
def process_datax(eqdfList, x=14, y=70):
    listx = []
    for dfx in eqdfList:
        if not dfx.empty:  # Check if DataFrame is not empty
            last_row = dfx.iloc[-1]

            # Check if MACD and RSI values are available for the last x rows
            if len(dfx) >= x:
                last_x_rows = dfx.iloc[-x:]
                # Calculate MACD
                macd = last_x_rows['close'].ewm(span=12).mean() - last_x_rows['close'].ewm(span=26).mean()
                macd_signal = macd.ewm(span=9).mean()
                last_row['macd'] = macd.iloc[-1]
                last_row['macd_signal'] = macd_signal.iloc[-1]

                # Calculate RSI
                delta = last_x_rows['close'].diff()
                gain = (delta.where(delta > 0, 0)).rolling(window=x).mean()
                loss = (-delta.where(delta < 0, 0)).rolling(window=x).mean()
                rs = gain / loss
                rsi = 100 - (100 / (1 + rs))
                last_row['rsi'] = rsi.iloc[-1]

                # Check MACD and RSI conditions
                if macd.iloc[-1] - macd_signal.iloc[-1] > 0 and rsi.iloc[-1] < x:
                    signal = 2  # Buy signal
                elif macd.iloc[-1] - macd_signal.iloc[-1] < 0 and rsi.iloc[-1] > y:
                    signal = 1  # Sell signal
                else:
                    signal = None
            else:
                signal = None

            # Append the signal to the DataFrame
            dfx['Signal'] = signal

            # Check if signal is generated
            if signal == 2:
                listx.append(last_row['symbol'])
            elif signal == 1:
                print("Sell signal generated for:", last_row['symbol'])
            else:
                print("No signal generated for:", last_row['symbol'])
        else:
            print("DataFrame is empty")

    return listx

```


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

def main():
    global flist
    eqdfList = fetch_candle_datax()
    flist = process_datax(eqdfList)

if __name__ == "__main__":
    main()

```