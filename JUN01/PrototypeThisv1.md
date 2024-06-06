
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Scraping Test Page</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
        }

        .search-box {
            margin-bottom: 20px;
        }

        input[type="text"] {
            padding: 8px;
            width: 200px;
        }

        input[type="submit"] {
            padding: 8px 16px;
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
        }
    </style>
</head>

<body>
    <h2>Search Page</h2>
    <form class="search-box" action="search" method="get">
        <input type="text" name="query" placeholder="Enter your search query">
        <input type="submit" value="Search">
    </form>
</body>

</html>
```

## script

```python


# !apt install chromium-chromedriver
# !pip install selenium
# !chromium-browser --product-version; chromedriver --version

# !apt-get install chromium-browser
# !snap install chromium

# import time, datetime
# import pandas as pd 
# import numpy as np
 
# from selenium import webdriver

# tradingview_creds = dict({'usrn': ['email@gamil.com'], 'passwd': ['usrusr']})
# x = pd.DataFrame(tradingview_creds)
# x.to_csv('cred.csv', index=False)
# usrn,passwd = pd.read_csv('cred.csv').iloc[0]['usrn'], pd.read_csv('cred.csv').iloc[0]['passwd']


from selenium.webdriver.chrome.options import Options
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By

url = "http://127.0.0.1:5500"
# # Path to your WebDriver (e.g., chromedriver, geckodriver)
# driver_path = "/path/to/your/webdriver"

# # Start the WebDriver
# driver = webdriver.Chrome(driver_path)


def get_driver():
  chrome_options = Options()
  chrome_options.add_argument('--no-sandbox')
  chrome_options.add_argument('--headless')
  chrome_options.add_argument('--disable--dev-shm-usage')
  driver = webdriver.Chrome(options=chrome_options)
  return driver


#python reSets special variable name having value main
if __name__ == "__main__":



    driver = get_driver()
    
    # Navigate to the test page
    driver.get(url)

    # Find the search box and input text
    search_box = driver.find_element(By.CSS_SELECTOR,'input[type="text"]')
    search_box.send_keys("test query")

    # Find the submit button and click
    submit_button = driver.find_element(By.CSS_SELECTOR,'input[type="submit"]')
    submit_button.click()

    # Get the updated URL
    updated_url = driver.current_url
    print("Updated URL:", updated_url)

    # Close the WebDriver
    driver.quit()




#https://www.selenium.dev/selenium/docs/api/py/webdriver/selenium.webdriver.common.by.html
```