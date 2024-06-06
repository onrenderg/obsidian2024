

## Setup Env Short

```bash
sudo apt install -y tmux apt install chromium-chromedriver
```

## Setup Env Long

```bash
sudo apt purge -y chromium-chromedriver
sudo apt autoremove


wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb

wget https://chromedriver.storage.googleapis.com/2.41/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
cd  chromedriver_linux64
sudo mv chromedriver /usr/bin/chromedriver
sudo chown root:root /usr/bin/chromedriver
sudo chmod +x /usr/bin/chromedriver


wget https://storage.googleapis.com/chrome-for-testing-public/123.0.6312.122/linux64/chrome-linux64.zip
unzip chrome-linux64.zip
cd chrome-linux64


wget https://storage.googleapis.com/chrome-for-testing-public/123.0.6312.122/linux64/chromedriver-linux64.zip
unzip chromedriver-linux64.zip
cd chromedriver-linux64
sudo mv chromedriver /usr/bin/chromedriver

sudo mv chromedriver /usr/bin/chromedriver
sudo chmod +x /usr/bin/chromedriver

wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb && sudo apt install ./google-chrome-stable_current_amd64.deb





```
## Testing env setup headless mode 

```python
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time

def get_driverx():
    # Define Chrome options
    chrome_options = Options()
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-dev-shm-usage')
    
    # Set the binary location of Chrome
    chrome_options.binary_location = '/teamspace/studios/this_studio/ddwn/chrome-linux64/chrome'

    # Set the path to ChromeDriver
    chromedriver_path = '/teamspace/studios/this_studio/ddwn/chromedriver-linux64/chromedriver'

    try:
        # Add ChromeDriver path as an argument
        chrome_options.add_argument(f"webdriver.chrome.driver={chromedriver_path}")

        # Initialize Chrome WebDriver with the specified options
        driver = webdriver.Chrome(options=chrome_options)
        
        return driver

    except Exception as e:
        print("Error:", e)

driver = get_driverx()
try:
    driver.get("https://www.google.com")
    print("done")

except Exception as e:
    print("Error:", e)

# Infinite loop to prevent script from terminating
while True:
    driver.get("https://www.google.com")
    time.sleep(1)  

```


### ScriptMapper

```python
import pandas as pd
import requests as re
import time 


def initmap():
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
    return NSElist

df = initmap()
df.to_csv("ms.csv")






from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.common.exceptions import TimeoutException, NoSuchElementException






def get_driverx():
    # Define Chrome options
    chrome_options = Options()
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-dev-shm-usage')
    
    # Set the binary location of Chrome
    chrome_options.binary_location = '/teamspace/studios/this_studio/ddwn/chrome-linux64/chrome'

    # Set the path to ChromeDriver
    chromedriver_path = '/teamspace/studios/this_studio/ddwn/chromedriver-linux64/chromedriver'

    try:
        # Add ChromeDriver path as an argument
        chrome_options.add_argument(f"webdriver.chrome.driver={chromedriver_path}")

        chrome_options.add_argument('--disable-dev-shm-usage')
        # Initialize Chrome WebDriver with the specified options
        driver = webdriver.Chrome(options=chrome_options)
        
        return driver

    except Exception as e:
        print("Error:", e)

def get_driver():
    chrome_options = Options()
    # chrome_options.add_argument('--no-sandbox')
    # chrome_options.add_argument('--headless')  # Experiment with headless mode if needed
    # chrome_options.add_argument('--disable-dev-shm-usage')
    driver = webdriver.Chrome(options=chrome_options)
    return driver









def retrieve_stock_url(stock_name):
    driver = get_driverx()
    try:
        driver.get("https://groww.in/charts/stocks/easy-trip-planners-ltd?exchange=BSE")
        wait = WebDriverWait(driver, 10)
        typebox = wait.until(EC.presence_of_element_located((By.XPATH, "//input[@id='chartSearch23']")))
        typebox.send_keys(stock_name)
        drop = wait.until(EC.element_to_be_clickable((By.XPATH, "//body/div[@id='__next']/div[@id='root']/div[@id='stockChartPage']/div[1]/div[1]/div[2]/div[1]/div[2]/div[2]/div[1]/div[1]/div[2]/div[1]/div[2]")))
        drop.click()
        time.sleep(4)



        updated_url = driver.current_url
    except TimeoutException as te:
        print(f"Timeout waiting for element for {stock_name}: {te}")
        updated_url = None
    except NoSuchElementException as ne:
        print(f"Element not found for {stock_name}. Check XPath: {ne}")
        updated_url = None
    except Exception as e:
        print(f"Error processing {stock_name}: {e}")
        updated_url = None
    finally:
        driver.quit()
    return updated_url


def retrieve_url(df):
    url_data = []  # Initialize an empty list to store dictionaries
    for index, row in df.iterrows():
        stock_sym = row["symbol"]
        stock_name = row["name"]
        url = retrieve_stock_url(stock_name)
        if url:  # Check if url is not None
            print(str(url))
            url_data.append({"symbol": stock_sym, "name": stock_name, "url": url})

    url_df = pd.DataFrame(url_data)  # Convert the list of dictionaries to a DataFrame
    return url_df

# Assuming you have your DataFrame df already defined
urlmap = retrieve_url(df)
urlmap.to_csv("urlmap.csv")


```
* make data needed by script self contained pure functionish 
*[[supabaseBucket]]
[[PrototypeThisv1]]

