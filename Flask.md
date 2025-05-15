*Flask* is a lightweight web framework, written in [[Python]].

A basic Flask application can look something like this:

```
from flask import Flask  # Import Flask

app = Flask(__name__)  # Make an instance of the Flask class

@app.route("/")  # Add a decorator to the app that for this endpoint, the function index should be used ran
def index():
    return "Hello World :D"  # And then the page displays what that function returns
```
