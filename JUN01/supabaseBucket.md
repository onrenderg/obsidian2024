
### Auth 

```python
# env.py
supabase_url = 'https://rsgkghynaaikzamospwt.supabase.co'

supabase_key = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6InJzZ2tnaHluYWFpa3phbW9zcHd0Iiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTcxNjE5NTM1MiwiZXhwIjoyMDMxNzcxMzUyfQ.I5DoKXmXVXlagAub_oMbxUqB8i4dvPGUNZRr50aVb0s'
```

## MainFile: Convert json in memory to json file uploaded directly to supabse buckt


```python
from flask import Flask, jsonify, request, send_file
import json
from supabase import create_client, Client
import os
# Initialize Flask app
app = Flask(__name__)

# Supabase configuration
from env import supabase_key, supabase_url
supabase: Client = create_client(supabase_url, supabase_key)


bucket_name = 'bucket-x'
file_name = 'data.json'
local_file_path = 'data.json'

@app.route('/write', methods=['POST'])
def write_data():
    data = {"message": "Hello, world!!"}
    response = supabase.storage.from_(bucket_name).upload(file_name, json.dumps(data).encode('utf-8'))
    if 'error' in response:
        return jsonify({"status": "error", "message": "Failed to write data", "error": response['error']}), 500
    else:
        return jsonify({"status": "success", "message": "Data written to Supabase bucket"}), 200

@app.route('/read', methods=['GET'])
def read_data():
    try:
        response = supabase.storage.from_(bucket_name).download(file_name)
        with open(local_file_path, 'wb+') as f:
            f.write(response)
        return send_file(local_file_path, as_attachment=True)
    except Exception as e:
        error_message = str(e)
        if '404' in error_message:
            return jsonify({"status": "error", "message": "File not found"}), 404
        else:
            return jsonify({"status": "error", "message": "Failed to read data"}), 500

if __name__ == '__main__':
    app.run(debug=True)
```