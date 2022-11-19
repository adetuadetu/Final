# My name is joan and this is my final project, Drivt (A blogging application)

## Contents 

1. Templates
2. Web forms
3. Databases
4. Email
5. Large application structure
6. How User authentication is implemented
7. How User Roles functions
8. Individual User Profiles
9. Adding Blog posts
10. Implementing Followers
11. Leaving User comments
12. Application Programming Interfaces
13. Testing 
14. Performane testing
15. Deployment using Docker

## 1. Templates

### What are templates?
A template is a file that contains all the information of a response which is rendered by the view functions within the views.py file e.g when the user clicks on the Home page icon within the application the Home page template will be trigured and the appropriate response will be determined by the business or presentation logic within the view function. When making this application i wanted to make my code as universal and easy to comprehend as possible, therefore i used the help of Flask's template engine called Jinja2.

### How is jinja2 useful
Jinja2 is an extremely useful and efficient template engine. One of the valuable features of Jinja2 is inheritence, the ability to save reocuring parts of the html code like the head contents into blocks that can be referenced in other templates using 

```Jinja2
{% extends "base.html" %}
```

instead of re-writing the whole head contents in each template. Below is an example of this Controled Structure

```Jinja2
{% block head %}
{{ super() }}
<link rel="shortcut icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
<link rel="icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='styles.css') }}">
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@20..48,100..700,0..1,-50..200" />
{% endblock %}
```

### Why integrte Bootstrap and how
Bootstrap is a great frontend framework which you can use to create user interface components that are compatible with all modern web browsers and all devices. Bootstrap is also a client side server so all the server needs to do is produce HTML responses that reference Bootstraps CSS and javaScript files.

Downloading the Flask-Bootstrap library is done using pip within the terminal window

```Pip
$ pip install flask-bootstrap 
```
Flask extensions are initialized at the same time as the application instance this is done in the __init__.py file below is an example of how this is done 

```Flask
from flask_bootstrap import Bootstrap
# ...
bootstrap = Bootstrap()
```
the bootstrap extension is initialized without the application instance so you can dynamically make configuration changes without the application being automatically initialized before any changes can be made. The solution is the delay the initialization of the appllication into a factory function called create_app() shown in the example below 

```Flask
def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)
```
The configuration settings stored within one of the classes defined in the config.py file can be imported straight into the application using the from_object() shown above which is available in app.config configuration object.

Once flask is initialized a bootstrap base template with all the Bootstrap files and general structure becomes available to the application by referencing it using 

```Flask
{% extends "bootstrap/base.html" %}
```


