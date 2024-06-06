
```python
#!/bin/python3

    # my_bbands = ta.bbands(dfstream.Close, length=15, std=1.5)
    # dfstream=dfstream.join(my_bbands)
import pandas as pd
import ta 
from flask import Flask, jsonify,request, redirect, session, render_template_string
from datetime import timedelta
# from apscheduler.schedulers.background import BackgroundScheduler


  
app = Flask(__name__)




def  calculate_indicators(df):
    
    # Calculate SMA's
    df['sma_12'] = ta.trend.sma_indicator(df['close'], window=12, fillna=False)
    df['sma_24'] = ta.trend.ema_indicator(df['close'], window=24, fillna=False)
    df['sma_50'] = ta.trend.ema_indicator(df['close'], window=50, fillna=False)
    df['sma_200'] = ta.trend.ema_indicator(df['close'], window=200, fillna=False)

    # Calculate RSI
    df['rsi'] = ta.momentum.rsi(df['close'], window=14, fillna=False)

    # Calculate MACD
    
    #Slow moving average: 26 days. Fast moving average: 12 days. Signal line: 9-day moving #average of the difference between fast and slow.

    df['macd_histogram'] = macd_data = ta.trend.macd_diff(df['close'], window_slow=26, window_fast=12, window_sign=9, fillna=False)
    df['macd_fast'] = ta.trend.macd_signal(df['close'], window_slow=26, window_fast=12, window_sign=9, fillna=False)
    df['macd_slow'] = ta.trend.macd(df['close'], window_slow=26, window_fast=12, fillna=False)

    return df








# Sample DataFrame
df = pd.read_csv('RHEL.csv')
df.drop(df.columns[0], axis=1, inplace=True)
df.columns = df.columns.str.lower()
df = df.drop('adj close', axis=1)


df['date'] = pd.to_datetime(df['date'], utc=True)
df['date'] = df['date'] + timedelta(hours=5, minutes=30)  # Specify UTC as the timezone
df['time'] = df['date'].astype(int) // 10**9

df = calculate_indicators(df)
current_index = 0


# # Sample DataFrame
# df = pd.read_csv('RHEL.csv')
dfx = df.head(1)  # Selecting the earliest row








def sidex(df):
    

    if df.iloc[-1]['close'] <= df.iloc[-2]['open']:
        return "Sell"
    elif df.iloc[-1]['close'] >= df.iloc[-2]['open']:
        return "Buy"

x = 0  # Global variable to track position status
entry_price = 0  # Global variable to store entry price
pct_change = 0
side = 0

def signalx(df):
    global x, entry_price, pct_change, side
    if x == 1:
        # pct_change = abs((df.iloc[-1]['close'] - entry_price) / entry_price * 100)
        pct_change = (df.iloc[-1]['close'] - entry_price) / entry_price * 100

        
        if side == 1:
            if pct_change <= -0.3:
                x = 0  # Close the position
                entry_price = 0
                return None  # No new signal as position is closed
    
        if side == 2:
            if pct_change >= 0.3 :
                x = 0  # Close the position
                entry_price = 0
                return None  # No new signal as position is closed
    
    # Check for new signals only if position is closed (x == 0)
    if x == 0:  
        if df.iloc[-1]['close'] <= df.iloc[-2]['open']:
            df.loc[df.index[-1], 'signal'] = '1'
            # df['signal'] = '1'
            x = 1
            side = 1
            entry_price = df.iloc[-1]['close']
            return "Sell"
        elif df.iloc[-1]['close'] >= df.iloc[-2]['open']:
            df.loc[df.index[-1], 'signal'] = '1'
            # df['signal'] = '2'
            x = 1
            side = 2
            entry_price = df.iloc[-1]['close']
            return "Buy"





# def sidex(df, current_candle, backcandles):
#     df_slice = df #df.reset_index().copy()

#     # Get the range of candles to consider
#     start = max(0, current_candle - backcandles)
#     end = current_candle
#     relevant_rows = df_slice.iloc[start:end] # iloc return ?

#     # Check if all EMA_fast values are below EMA_slow values
#     # if all(relevant_rows["EMA_fast"] < relevant_rows["EMA_slow"]):
#     if all(relevant_rows["sma_50"] < relevant_rows["sma_200"]):
#         return 1 # Short

#     # elif all(relevant_rows["EMA_fast"] > relevant_rows["EMA_slow"]):
#     elif all(relevant_rows["sma_50"] > relevant_rows["sma_200"]):
#         return 2 #Long
#     else:
#         return 0

# # side = sidex(dfx)

# def signalx(df, current_candle, backcandles):
#     if (sidex(df, current_candle, backcandles)==1
#         # and df.Close[current_candle]<=df['BBL_15_1.5'][current_candle]
#         and df.Close[current_candle]>=df['sma_12'][current_candle]
#         #and df.rsi[current_candle]>40
#         ): #Short
#             return 1

#     if (sidex(df, current_candle, backcandles)==2
#         # and df.Close[current_candle]>=df['BBU_15_1.5'][current_candle]
#         and df.Close[current_candle]<=df['sma_12'][current_candle]
#         #and df.rsi[current_candle]<60
#         ): #Long
#             return 2

#     return 0

def order(side, signal,df):
    print(f"{side}")
    print(f"{signal}")
    print(f"Close:{df.iloc[-1]['close']} ::: Open:{df.iloc[-2]['open']}")
    print(f"%CloseFromEntry:{pct_change}")
    print("#######################################################################")


def get_kline(n):
    # Sample implementation of kline returning consecutive rows
    new_data_df = df.head(n) 
    return new_data_df  

def kline(index):
    # Sample implementation of kline returning consecutive rows
    new_data = df.iloc[index]  # Get the row corresponding to the current index
    return pd.DataFrame([new_data])  # Convert the row to a DataFrame

current_index = 0

def  range():
    global current_index, dfx

    new_df = kline(current_index+1) 
    dfx = pd.concat([dfx, new_df])
    # dfx = dfx.fillna(0)

    print(dfx)   

    side = sidex(dfx)
    signal = signalx(dfx)
    order(side, signal,df=dfx)
    dfx = dfx.fillna(0)

    # current_index = current_index + 1

    # return  dfx[-1].to_dict() #dict of series latest 
    # return dfx.iloc[current_index].to_dict()
    data_for_index = dfx.iloc[current_index].to_dict()
    current_index += 1
    
    return data_for_index

    # return  dfx.iloc[-1].to_dict() #dict of series latest 


@app.route('/stream')
def stream():
    global current_index, df    
    if current_index >= len(df):
        return jsonify({'message': 'Data stream ended'}), 200
    
    dictx = range()

    return jsonify(dictx), 200



index_html ="""
<!DOCTYPE html>
<html lang="en">

<head>
    <style>
        #tvchart {
            position: relative;
            width: 100vw;
            height: 100vh;
        }
    </style>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>TV Indicators</title>
</head>

<body>
    <div id="tvchart"></div>
    <script https://unpkg.com/lightweight-charts/dist/lightweight-charts.standalone.production.js>

    </script>
    
    <script>
const {
    log,
    error
} = console;

const getData = async () => {
    const resp = await fetch('http://127.0.0.1:5001/stream');
    const data = await resp.json();
    return data;
};

const renderChart = async () => {
    const chartProperties = {
        timeScale: {
            timeVisible: true,
            secondsVisible: true,
        },
    };

    const domElement = document.getElementById('tvchart');
    const chart = LightweightCharts.createChart(domElement, chartProperties);
    const candleseries = chart.addCandlestickSeries();


    const sma_200_series = chart.addLineSeries({
        color: 'blue',
        lineWidth: 2
    });
    const sma_50_series = chart.addLineSeries({
        color: 'black',
        lineWidth: 1
    });
    const sma_24_series = chart.addLineSeries({
        color: 'red',
        lineWidth: 1
    });
    const sma_12_series = chart.addLineSeries({
        color: 'yellow',
        lineWidth: 1
    });


    const rsi_series = chart.addLineSeries({
        color: 'purple',
        lineWidth: 1,
        pane: 1,
    });

    const macd_fast_series = chart.addLineSeries({
        color: 'blue',
        lineWidth: 1,
        pane: 2,
    });
    const macd_slow_series = chart.addLineSeries({
        color: 'red',
        lineWidth: 1,
        pane: 2,
    });
    const macd_histogram_series = chart.addHistogramSeries({
        pane: 2,
    });

    let markers = []; // Array to store markers

    setInterval(async () => {
        const data = await getData();
        console.log(data);





        // Update candlestick series with new data
        candleseries.update({
            time: data.time,
            open: data.open,
            high: data.high,
            low: data.low,
            close: data.close
        });

        // Update each series with new data
        sma_200_series.update({
            time: data.time,
            value: data.sma_200
        });
        // Update each series with new data
        sma_50_series.update({
            time: data.time,
            value: data.sma_50
        });
        // Update each series with new data
        sma_24_series.update({
            time: data.time,
            value: data.sma_24
        });
        // Update each series with new data
        sma_12_series.update({
            time: data.time,
            value: data.sma_12
        });


        rsi_series.update({
            time: data.time,
            value: data.rsi
        });

        
        macd_fast_series.update({
            time: data.time,
            value: data.macd_fast
        });
        macd_slow_series.update({
            time: data.time,
            value: data.macd_slow
        });
        macd_histogram_series.update({
            time: data.time,
            value: data.macd_histogram,
            color: data.macd_histogram > 0 ? 'green' : 'red'
        });

        // Marker logic (adapt based on your signal generation)
        if (data.signal === '1') { // Assuming '1' represents a sell signal
            markers.push({
                time: data.time,
                position: 'aboveBar',
                color: 'red',
                shape: 'arrowDown',
                text: 'SELL'
            });
        } else if (data.signal === '2') { // Assuming '2' represents a buy signal
            markers.push({
                time: data.time,
                position: 'belowBar',
                color: 'green',
                shape: 'arrowUp',
                text: 'BUY'
            });
        }

        candleseries.setMarkers(markers); // Set all markers
    }, 1000); // Fetch data every 1 second
};

renderChart();


    </script>
</body>

</html>
"""
@app.route('/')
def index():
    return render_template_string(index_html)

def get_candles_frame(n):
    candles = get_raw_kline(n)# get_candles(n) # Return list of dict=rows /get_kline(n)
    # dfstream = pd.DataFrame(columns=['Open','Close','High','Low'])
    df = candles 
# Raw data processing     
    # i=0
    # for candle in candles:
    #     dfstream.loc[i, ['Open']] = float(str(candle.bid.o))
    #     dfstream.loc[i, ['Close']] = float(str(candle.bid.c))
    #     dfstream.loc[i, ['High']] = float(str(candle.bid.h))
    #     dfstream.loc[i, ['Low']] = float(str(candle.bid.l))
    #     i=i+1



    # dfstream['Open'] = dfstream['Open'].astype(float)
    # dfstream['Close'] = dfstream['Close'].astype(float)
    # dfstream['High'] = dfstream['High'].astype(float)
    # dfstream['Low'] = dfstream['Low'].astype(float)
    df.drop(df.columns[0], axis=1, inplace=True)
    df.columns = df.columns.str.lower()
    df = df.drop('adj close', axis=1)

    df['date'] = pd.to_datetime(df['date'], utc=True)
    df['date'] = df['date'] + timedelta(hours=5, minutes=30)  # Specify UTC as the timezone
    df['time'] = df['date'].astype(int) // 10**9

    
    # dfstream["ATR"] = ta.atr(dfstream.High, dfstream.Low, dfstream.Close, length=7)
    # dfstream["EMA_fast"]=ta.ema(dfstream.Close, length=30)
    # dfstream["EMA_slow"]=ta.ema(dfstream.Close, length=50)
    # dfstream['RSI']=ta.rsi(dfstream.Close, length=10)
    # my_bbands = ta.bbands(dfstream.Close, length=15, std=1.5)
    # dfstream=dfstream.join(my_bbands)
    df = calculate_indicators(df)

    df = df.fillna(0)
    dfstream = df

    return dfstream    

# def trading_job():
#     dfstream = get_candles_frame(200)
#     signal = signalx(dfstream, len(dfstream)-1/dfstream[-1].index, 7) # previous candle signal current candle still opened
        
#     # Order
#         #Sell
#     if signal == 1 and 'percentage_chage_series.iloc[-1] >50:' /count_opened_t [rades() == 0 and spread<max_spread:
#         mo = MarketOrderRequest(instrument="EUR_USD", units=-3000, takeProfitOnFill=TakeProfitDetails(price=TPSell).data, stopLossOnFill=StopLossDetails(price=SLSell).data)
#         r = orders.OrderCreate(accountID, data=mo.data)
#         rv = client.request(r)
#         print(rv) #Flask msg
#     #Buy
    
#     elif signal == 2 and '% from open'  count_opened_trades() == 0 and spread<max_spread:
#         mo = MarketOrderRequest(instrument="EUR_USD", units=3000, takeProfitOnFill=TakeProfitDetails(price=TPBuy).data, stopLossOnFill=StopLossDetails(price=SLBuy).data)
#         r = orders.OrderCreate(accountID, data=mo.data)
#         rv = client.request(r)
#         print(rv) #Flask msg


# sched = BackgroundScheduler(daemon=True)

# sched.add_job(trading_job, 'cron', day_of_week='*', hour='*', minute='*', timezone='Asia/Kolkata')

# sched.start()



if __name__ == '__main__':
    app.run(debug=False, port=5001)

```
[[flask_socketio]]
