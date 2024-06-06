

```python

#  Initializing Supabase client
## `create_client()` method.

# By default, Supabase projects return a maximum of 1,000 rows.
## https://supabase.com/dashboard/project/_/settings/api


from supabase import create_client
import json
API_URL = 'https://pidvimlpfbtztwitweml.supabase.co'
API_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBpZHZpbWxwZmJ0enR3aXR3ZW1sIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MTE2NjQzOTYsImV4cCI6MjAyNzI0MDM5Nn0.6bcjCb0xR4kOY1BDPJOCRCrM5V-MzD4UrO2rzRNkLeQ'
supabase = create_client(API_URL, API_KEY)
supabase


# test  
## Each record needs to be a dictionary containing all the information
### use the “insert()
### , we must pass in a list of dictionaries containing all the records’ data


data = {
    'id': 1,
    'name': 'Vishnu',
    'age': 22,
    'country': 'India',
    'programming_languages': json.dumps(['C++', 'python', 'Rust'])
}
supabase.table('demo-database').insert(data).execute() # inserting one record

# data, count = supabase.table('countries') .insert({"id": 1, "name": "Denmark"}) .execute()
  
# response = supabase.table('demo-database').select("*").execute()
# print(response)
# https://supabase.com/docs/reference/python/upsert
# https://github.com/supabase-community/supabase-**py**
```

```
```python


import yfinance as yf
from supabase import create_client, Client
from flask import Flask, request, jsonify

# Supabase connection details (replace with your actual values)
SUPABASE_URL = 'https://pidvimlpfbtztwitweml.supabase.co'
SUPABASE_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBpZHZpbWxwZmJ0enR3aXR3ZW1sIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MTE2NjQzOTYsImV4cCI6MjAyNzI0MDM5Nn0.6bcjCb0xR4kOY1BDPJOCRCrM5V-MzD4UrO2rzRNkLeQ'
# Create Supabase client
supabase: Client = create_client(SUPABASE_URL, SUPABASE_KEY)

# Flask app initialization
app = Flask(__name__)

@app.route("/insert_data", methods=["GET"])
def insert_data():
    try:
        # Get ticker list from request body
        ticker_list = ["AAPL", "TSLA", "GOOG"] #request.json.get("tickers")

        # Validate ticker list
        if not ticker_list or not isinstance(ticker_list, list):
            return jsonify({"error": "Invalid ticker list"}), 400

        # Download data for the first ticker to get the schema
        result = yf.download(
            tickers=ticker_list[0],
            start="2022-03-10",
            end="2022-03-10",
            prepost=True,
            progress=False
        )

        # Create table schema based on DataFrame columns
        table_schema = [
            {"name": col, "type": "text" if result[col].dtype == "object" else "numeric"}
            for col in result.columns
        ]

        # Create the table if it doesn't exist
        response = supabase.table("test").create(table_schema).execute()
        if response.error:
            return jsonify({"error": response.error.message}), 500

        # Iterate through tickers and insert data
        for i, ticker in enumerate(ticker_list):
            # Download data using yfinance
            result = yf.download(
                tickers=ticker,
                start="2022-03-10",
                end="2022-03-10",
                prepost=True,
                progress=False
            )

            # Convert DataFrame to list of dictionaries
            data = result.to_dict(orient="records")

            # Insert data into Supabase table "test"
            response = supabase.table("test").insert(data).execute()

            if response.error:
                return jsonify({"error": response.error.message}), 500

        return jsonify({"message": "Data inserted successfully"})

    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == "__main__":
    app.run(debug=True)
```

## Get

```python


from supabase import create_client
import json
API_URL = 'https://pidvimlpfbtztwitweml.supabase.co'
API_KEY = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InBpZHZpbWxwZmJ0enR3aXR3ZW1sIiwicm9sZSI6ImFub24iLCJpYXQiOjE3MTE2NjQzOTYsImV4cCI6MjAyNzI0MDM5Nn0.6bcjCb0xR4kOY1BDPJOCRCrM5V-MzD4UrO2rzRNkLeQ'
supabase = create_client(API_URL, API_KEY)



# data0 = {
#     'id': 1,
#     'name': 'Vishnu',
#     'age': 22,
#     'country': 'India',
#     'programming_languages': json.dumps(['C++', 'python', 'Rust'])
# }
# supabase.table('test').insert(data).execute() # inserting one record

# data = [
#     {
#         'id': 2,
#         'name': 'Prakash',
#         'age': 37,
#         'country': 'India',
#         'programming_languages': json.dumps(['C#', 'web assembly'])
#     },
#     {
#         'id': 3,
#         'name': 'Arjun',
#         'age': 29,
#         'country': 'Germany',
#         'programming_languages': json.dumps(['python', 'nodejs', 'Rust'])
#     },
#     {
#         'id': 4,
#         'name': 'Sanjay',
#         'age': 19,
#         'country': 'India',
#         'programming_languages': json.dumps(['python'])
#     },
#     {
#         'id': 5,
#         'name': 'Ram',
#         'age': 44,
#         'country': 'India',
#         'programming_languages': json.dumps(['python', 'Go'])
#     }
# ]
# supabase.table('test').insert(data).execute() # inserting multiple records

response = supabase.table('test').select("*").execute()
print(response)

```