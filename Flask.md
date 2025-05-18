<dfn>Flask</dfn> is a lightweight framework for building software for [[TheWeb]], written in [[Python]].

A basic Flask application can look something like this:

```python
from flask import Flask

# Create an instance of the Flask class
app = Flask(__name__)

# Add a decorator to the app which indicates that for this route,
# the function index should be used ran
@app.route("/")
def index():
     # And then the page displays what that function returns
    return "hewwo wowld :3"
```
