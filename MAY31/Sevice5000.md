scp -i ./awslighsail.pem /home/caspi/Desktop/2024/MAY31/masterurlmap.csv ubuntu@13.126.121.104:/home/ubuntu/service5000


```python

from flask import Flask, render_template_string, jsonify, request

import random

  

app = Flask(__name__)

  

# Sample function to generate random links

def genLink():

links = ["https://example.com/page1", "https://example.com/page2", "https://example.com/page3"]

random.shuffle(links)

return {"Links": links}

  

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

app.run(debug=True)

# curl -X POST http://localhost:5000/api1

# curl -i -X POST http://localhost:5000/api1
```

```bash
sudo apt install -y python3-pip
pip3 install -r req*


echo -e "flask\nwebsocket-client\nsmartapi-python\npyotp\nlogzero\nstreamlit\nFlask\ndatetime\npandas\nnumpy" > requirements.txt
```

[[MAY31/PyScreener|PyScreener]]
