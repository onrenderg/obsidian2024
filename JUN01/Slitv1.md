

```txt
lightweight-charts

pandas

streamlit

yfinance

qtpy

#PyGObject

#PyQt5==5.15.10

pywebview==4.3.3

#pygobject
```


```python
import pandas as pd
from lightweight_charts.widgets import StreamlitChart
import streamlit as st
import yfinance as yf
from datetime import datetime, timezone, timedelta
            



def calculate_sma(df, period: int = 50):
    return pd.DataFrame({
        'time': df['time'],
        f'SMA {period}': df['close'].rolling(window=period).mean()
    }).dropna()


def calculate_rsi(df):
    # Calculate RSI only using the relevant columns
    df['time'] = (pd.to_datetime(df['time']).view('int64') // 10 ** 9)

    # Calculate the price change
    delta = df['close'].diff()

    # Define the time period for RSI calculation (usually 14 days)
    window_length = 14

    # Calculate gain and loss
    gain = (delta.where(delta > 0, 0)).rolling(window=window_length).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=window_length).mean()

    # Calculate Relative Strength
    rs = gain / loss

    # Calculate RSI
    rsi = 100 - (100 / (1 + rs))

    # Convert 'time' back to datetime object
    df['time'] = pd.to_datetime(df['time'], unit='s', utc=True)

    # Add RSI column to the dataframe
    df['RSI'] = rsi

    return df


def calculate_macd(df, short_window=12, long_window=26, signal_window=9):
    # Calculate short-term and long-term exponential moving averages
    short_ema = df['close'].ewm(span=short_window, adjust=False).mean()
    long_ema = df['close'].ewm(span=long_window, adjust=False).mean()

    # Calculate MACD line
    macd_line = short_ema - long_ema

    # Calculate signal line
    signal_line = macd_line.ewm(span=signal_window, adjust=False).mean()

    # Add MACD and signal line columns to the dataframe
    df['MACD'] = macd_line
    df['Signal_Line'] = signal_line

    return df


df = pd.read_csv('ohlcv.csv')
print(df)
df = df.rename(columns={"timestamp": "time", "Open": "open", "High": "high", "Low": "low", "Close": "close","Volume": "volume"})
# Convert the 'timestamp' column to a datetime object
df['time'] = pd.to_datetime(df['time'], utc=True)
df['time'] = df['time'] - timedelta(hours=5)

# Calculate RSI
df = calculate_rsi(df)

# Calculate MACD
df = calculate_macd(df)




# Set Streamlit page configuration to full width
# st.set_page_config(layout="wide", initial_sidebar_state="expanded")
# Set the theme properties
st.set_page_config(
    layout="wide", initial_sidebar_state="expanded",
    page_title="Your App Title",
    page_icon=":chart_with_upwards_trend:"
)

# Customize theme colors and fonts
st.markdown(
    """
    <style>
    body {
        background-color: #F0FF33;  /* Background color */
        color: #03080C;  /* Text color */
        font-family: sans-serif;  /* Font */
    }
    .stButton > button {
        background-color: #F36295;  /* Primary button color */
    }
    .css-1egp28t {
        background-color: #3183D1;  /* Secondary button color */
    }
    </style>
    """,
    unsafe_allow_html=True,
)

st.sidebar.header("Financial Chart")

# Input for selecting a stock symbol
ticker = st.sidebar.text_input("Stock Symbol (e.g., AAPL)", value="AAPL").upper()

# Date range selection
start_date = st.sidebar.date_input("Start Date", pd.to_datetime("2022-01-01"))
end_date = st.sidebar.date_input("End Date", pd.to_datetime("2022-12-31"))  

# Interval selection
intervalv = st.sidebar.text_input("Interval (e.g., '1d' for daily, '1h' for hourly, etc.)", value="1d")

# Fetch data from Yahoo Finance
# data = yf.download(ticker, start=start_date, end=end_date, interval=intervalv)


chart = StreamlitChart(width=1099, height=498,inner_width=1, inner_height=0.5, toolbox=True)


# Plot the chart
chart.legend(visible=True)
chart.time_scale(visible=False)

#SMA
line0 = chart.create_line('SMA 50', color='white')
sma_data0 = calculate_sma(df, period=50)
line0.set(sma_data0)

line1 = chart.create_line('SMA 200', color='blue')
sma_data1 = calculate_sma(df, period=200)
line1.set(sma_data1)

# Subchart for RSI
chart2 = chart.create_subchart(width=1, height=0.2, sync=True)
line_rsi = chart2.create_line()

# Subchart for MACD lines
chart3 = chart.create_subchart(width=1, height=0.3, sync=True)
line_macd = chart3.create_line(color="blue")  # MACD line in blue
line_signal = chart3.create_line(color="red")  # Signal line in red

df_rsi = pd.DataFrame({'time': df['time'], 'value': df['RSI']})
df_macd = pd.DataFrame({'time': df['time'], 'value': df['MACD']})
df_signal = pd.DataFrame({'time': df['time'], 'value': df['Signal_Line']})

chart.set(df)
line_rsi.set(df_rsi)

line_macd.set(df_macd)
line_signal.set(df_signal)

# Watermark
chart.watermark(ticker)

chart.load()



    # Add a button to make the chart fullscreen
if st.button("Fullscreen Chart"):
        # Add CSS to target the iframe with data-testid="stIFrame" and apply a class
        st.markdown("""
        <style>
        iframe[data-testid="stIFrame"] {
            position: fixed;
            top: 0;
            left: 0;
            bottom: 0;
            right: 0;
            width: 100%;
            height: 100%;
            border: none;
            margin: 0;
            padding: 0;
            overflow: hidden;
            z-index: 999999;
        }
        </style>
        """, unsafe_allow_html=True)

```

[[Dataforthis_kldsjfkl]]
