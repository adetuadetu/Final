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

```Python
from flask_bootstrap import Bootstrap
# ...
bootstrap = Bootstrap()
```
the bootstrap extension is initialized without the application instance so you can dynamically make configuration changes without the application being automatically initialized before any changes can be made. The solution is the delay the initialization of the appllication into a factory function called create_app() shown in the example below 

```Python
def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)
```
The configuration settings stored within one of the classes defined in the config.py file can be imported straight into the application using the from_object() shown above which is available in app.config configuration object.

Once flask is initialized a bootstrap base template with all the Bootstrap files and general structure becomes available to the application, the jinja2 (extends) directive implements the template inheritence by referencing  

```Python
{% extends "bootstrap/base.html" %}
```

the template provided by Flask-Bootstrap creates a skeleton web page that includes all CSS and JavaScript files from Bootstrap 

### Custom error pages
Within Flask you can define custom error pages that are consistent with the design of the web application using an app.errorhandler decorater which is similar to a view function. This is in the below example

```Python
@main.app_errorhandler(403)
def forbidden(e):
    if request.accept_mimetypes.accept_json and \
            not request.accept_mimetypes.accept_html:
        response = jsonify({'error': 'forbidden'})
        response.status_code = 403
        return response
    return render_template('403.html'), 403
```
error handlers produce templates like view function but they also return a numeric error value which can be implemented as an argument ass shown above 

### Static files 
Within Flask you can reference to static files which are saved in the subdirectory called static. Shown in the example below.

```Python
<link rel="shortcut icon" href="{{ url_for('static', filename='favicon.ico') }}" type="image/x-icon">
```

## 2. Web Forms

### How to implement web form in flask 
Forms generally need to have a flow of data going to the server in the sense of a users email address or password and a flow of data going from the server to the user like a confirmation email after the user has registered. All this can be done easier using the Flask-WTF extension which is installed using pip as shown below.

```Pip
$ pip install flask-wtf
```

unlike other flask extensions Flask-WTF does not need to be initialized on the application level but it expects the application to have a secret key configured. This key is used to secure from any cross-site request forgery (CSRF), making it more secure for users. This is shown below.

```Python
class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY')
```

Flask-WTF creates security tokens for all the forms and stores them in the user session, which is protected with a cryptographic signature produced from the secret key

### What are form classes 
When using Flask-WTF each style of web forms is represented in the server by a class. Depending on the class, the fields in the form will be different and represented by an object. Each field object has one or more validators attached making sure that any data written into the fields are valid. Below is an example.

```Python
class NameForm(FlaskForm):
    name = StringField('What is your name?', validators=[DataRequired()])
    submit = SubmitField('Submit')
```

### How to render forms
Instead of rendering forms manually we can use a great helper function wich is provided by the Flask-Bootstrap extension. Within the template file as shown below.

```Python
{% import "bootstrap/wtf.html" as wtf %}
{{ wtf.quick_form(form) }}
```




