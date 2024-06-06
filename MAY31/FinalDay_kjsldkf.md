************
## Module
* touch __init__.py

```bash
!echo -e "flask\nwebsocket-client\nsmartapi-python\npyotp\nlogzero\nstreamlit\nFlask\ndatetime\npandas\nnumpy" > requirements.txt

  

!pip3 install -r re*

```


```python
def modul():


    ## Import and auth

    from SmartApi import SmartConnect #or from SmartApi.smartConnect import SmartConnect
    from pyotp import TOTP
    from logzero import logger
    import pandas as pd
    import requests as rq
    import time
    from datetime import datetime, date, timedelta

    # Telegram notification function
    def rTBotProcess(bot_msg):
        bot_token = "7139273659:AAFchhNEVEi9cpOl6hRfy2J_1r50tnjq_3Q"
        bot_chatID = "5415845501"
        send_text = "https://api.telegram.org/bot" + bot_token + "/sendMessage?chat_id=" + bot_chatID + "&parse_mode=Markdown&text=" + bot_msg
        res = rq.get(send_text)
        print(res)

    totp_key = 'T2PX34ABOGB3VZIVENMUAOJ5PQ'
    api_key = 'kSJIOphE '
    username = 'P547740'
    pin = '8894'

    obj = SmartConnect(api_key=api_key)
    data = obj.generateSession(username, pin, TOTP(totp_key).now())

    ## GetNSETickList

    url = "https://margincalculator.angelbroking.com/OpenAPI_File/files/OpenAPIScripMaster.json"
    d = rq.get(url).json()

    tk_df = pd.DataFrame.from_dict(d)
    tk_df['expiry'] = pd.to_datetime(tk_df['expiry'])
    dfx = tk_df.astype({'strike': float})

    # Specify the unique value you want to search for
    unique_value1 = 'NSE'
    unique_value2 = 'BSE'

    # Use boolean indexing to filter rows with the specified unique value
    filtered_rows = dfx[(dfx['exch_seg'] == unique_value2) | (dfx['exch_seg'] == unique_value1) & dfx['symbol'].str.endswith('-EQ')]
    filtered_rows = filtered_rows[filtered_rows['exch_seg'] == unique_value1]
    NSElist = filtered_rows
    dfxx = NSElist

    # DatetimeLogicForAutomaticDownload
    def get_previous_working_day(start_date, days_back):
        current_date = start_date
        while days_back > 0:
            current_date -= timedelta(days=1)
            if current_date.weekday() < 5:  # Monday to Friday are considered working days
                days_back -= 1
        return current_date

    today_date = date.today()
    from_time = get_previous_working_day(today_date, 6)  # 6 working days before today
    to_time = today_date

    from_time_str = from_time.strftime('%Y-%m-%d 09:15')
    to_time_str = to_time.strftime('%Y-%m-%d 14:30')

    # Download rDataFrames
    def getCandleDatax(symbolInfo):
        try:
            historicParam={
                "exchange": symbolInfo.exch_seg,
                "symboltoken": symbolInfo.token,
                "interval": "ONE_MINUTE",
                "fromdate": from_time_str,
                "todate": to_time_str
            }
            res_json = obj.getCandleData(historicParam)
            columns = ['timestamp','open','high','low','close','volume']
            df = pd.DataFrame(res_json['data'], columns=columns)
            df['timestamp'] = pd.to_datetime(df['timestamp'], format='%Y-%m-%dT%H:%M:%S%z')
            df['symbol'] = symbolInfo.symbol
            print(f"Done for {symbolInfo.symbol}")
            time.sleep(0.4)
            return df
        except Exception as e:
            print(f"Historic Api failed: {e} {symbolInfo.symbol}")

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

    rTBotProcess(f"started")

    eqdfList = fetch_candle_datax()

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

    def process_datax(eqdfList):
        listx = []
        for dfx in eqdfList:
            if not dfx.empty:  # Check if DataFrame is not empty
                last_row = dfx.iloc[-1]
                if last_row['close'] > last_row['open']:
                    signal = 2
                elif last_row['close'] < last_row['open']:
                    signal = 1
                else:
                    signal = None

                dfx['Signal'] = signal

                if signal == 2:
                    listx.append(last_row['symbol'])
                else:
                    print("No signal generated for:", last_row['symbol'])
            else:
                print("DataFrame is empty")

        return listx

    slist = process_datax(eqdfList)
    print(slist)

    # OutputX
    df1 = pd.read_csv("./masterurlmap.csv")

    def get_url_for_symbols(symbol_list, df):
        urllist = []
        for symbol in symbol_list:
            try:
                row = df[df['symbol'] == symbol]
                url = row['url'].values[0]
                urllist.append(url)
            except IndexError:
                print(f"No URL found for symbol: {symbol}")
                urllist.append(None)
        return urllist

    url_list = get_url_for_symbols(slist, df=df1)
    print(url_list)

    # Save the url_list to a file in JSON format
    import json

    with open('./url_list.json', 'w') as f:
        json.dump(url_list, f)

    # import json
    # import random

    # def genLink():
    #     with open("url_list.json", "r") as file:
    #         data = json.load(file)
    #     random.shuffle(data)
    #     return {"Links": data}

    # Send a Telegram message
    site = "www.cloudforge.cloud"
    rTBotProcess(f"URL list has been updated and saved to file.{site}")

	return "Done"
stringx = modul()
print(strinx)

```




```python
from flask import Flask, render_template_string, jsonify, request
import random, os 
import json
from appmain import modul

app = Flask(__name__)


def genLink():
    with open("url_list.json", "r") as file:
        data = json.load(file)
    random.shuffle(data)
    return {"Links": data}

# Default route using render_template_string
@app.route('/')
def home():
    
    
    html_stringx = """
    <h1>Home Page</h1>
    <ul>
    {% for link in links %}
        <li><a href="{{ link }}" target="_blank">Link</a></li>
    {% endfor %}
    </ul>
    """

    html_string = """
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Clickable Links</title>
    <style>
        /* CSS for changing link color when clicked */
        ul {
            list-style-type: none;
            padding: 0;
        }
        li {
            margin-bottom: 5px;
        }
        a {
            color: black;
            text-decoration: none;
            transition: color 0.3s ease; /* Smooth transition for color change */
        }
        a:hover {
            color: red; /* Color change on hover */
        }
        .clicked {
            color: red !important; /* Change color to green when link is clicked */
        }
    </style>
</head>
<body>
    <h1>Clickable Links</h1>
    <ul id="link-list">
        {% for link in links %}
            <li><a href="{{ link }}" target="_blank" onclick="markClicked(this)">{{ loop.index }}. Link</a></li>
        {% endfor %}
    </ul>

    <script>
        // JavaScript function to mark clicked link
        function markClicked(link) {
            link.classList.add("clicked"); // Add 'clicked' class to clicked link
        }
    </script>
</body>
</html>
    """
    return render_template_string(html_string, links=genLink()["Links"])


# API route to trigger module file
@app.route('/api0', methods=['POST'])
def trigger_gen_link():
    if request.method == 'POST':
        links_dict = modul()
        return jsonify({"Key": "RunningLogic"})



# API route to trigger genLink() function
@app.route('/api1', methods=['POST'])
def trigger_gen_link():
    if request.method == 'POST':
        links_dict = genLink()
        return jsonify({"Links": links_dict["Links"]})

# API route returning JSON response
@app.route('/api2')
def api2():
    return jsonify({"message": "hi"})

if __name__ == '__main__':
    app.run(debug=True,port=5000)
# curl -X POST http://localhost:5000/api1
# curl -i -X POST http://localhost:5000/api1

```



```bash
curl -X POST http://localhost:5000/api1
curl -i -X POST http://localhost:5000/api1
```