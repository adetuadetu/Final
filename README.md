# DRIVT a blogging application



#### Description:

## Contents 

1. Templates
2. Web forms
3. Databases
4. Email
5. Large application structure
6. How User authentication is implemented
7. Deployment using Docker



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

### How to handle forms in view functions
For example the index() view function has to render the from and it also had to recieve the data entered by the user. Example below.

```Python
@main.route('/', methods=['GET', 'POST'])
def index():
    form = PostForm()
    if current_user.can(Permission.WRITE) and form.validate_on_submit():
        post = Post(body=form.body.data, author=current_user._get_current_object())
        db.session.add(post)
        db.session.commit()
        return redirect(url_for('.index'))
    page = request.args.get('page', 1, type=int)
    show_followed = False
    if current_user.is_authenticated:
        show_followed = bool(request.cookies.get('show_followed', ''))
    if show_followed:
        query = current_user.followed_posts
    else:
        query = Post.query
    pagination = query.order_by(Post.timestamp.desc()).paginate(page, per_page=current_app.config['FLASKY_POSTS_PER_PAGE'], error_out=False)
    posts = pagination.items
    return render_template('index.html', form=form, posts=posts, show_followed=show_followed, pagination=pagination)
```

In the example above, the method argument is used as a handler for POST or GET separately or together. When not specified the view function is registered only to GET requests.

### Redirects and User Sessions
Usually if you enter any data into a form and send it, when you refresh the page all your submited data will be sent again. We obviously dont want that so the way to tackle this issue is to respond to POST requests with a redirect which is essentially a URL for the page.

In order for the application to remember the users name, for example instead of storing that data within a local variable we will store it in a session.

### Message Flashing
Its good to let the user know if the form request has worked or theyve forgotten to fill something out thats what message flashing is good for. Flask includes the functionality as a core feature this is done as show in the example below.

```Python
flash('Your profile has been updated.')
```
but this flash message within view.py isnt enough we also have to reference it in the template file like shown below.

```Python
{% block content %}
<div class="container">
    {% for message in get_flashed_messages() %}
    <div class="alert alert-warning">
        <button tape="button"class="close" data-dismiss="alert">&times;</button>
        {{ message }}
    </div>
{% endfor %}
```


## 3. Databases

### What are databases ?
Databases store data from applications in an organized way. The application can then issue queries to retrieve specific data. Most common databases stemming from a relational model called SQL databases aka Structured Query Langauge. There are also different types of databases like document-oriented or key-value databases aka NoSQL databases.

### What database style did i use and why
I used a relational database called MySQL and from this im using the SQLAlchemy package. SQLAlchemy allows me to work at a higher level with regular Python objects instead of databse entities

### Getting started 
Firstly i have to download the flask-sqlalchemy extension

```Pip
$ pip install flask-sqlalchemy
```

The url of the application database has to be configured as the key

```Python
class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite')
```

in the flask config.py file 

### Model definition
The term model is used to describe the persistant entities used by the application an example of this is a python class with attributes the match the columns of a corresponding database. The database instance from FLASK-SQLAlchemy comes with a base class for models as well as helper classes and functions. Here is an example of my Roles and User models.

```Python
class Role(db.Model):
    __tablename__ = 'roles'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), unique=True)
    default = db.Column(db.Boolean, default=False, index=True)
    permissions = db.Column(db.Integer)
    users = db.relationship('User', backref='role', lazy='dynamic')
```

### Relationships between tables
Relational databases are great because you can reference other databases using foreign key. In the example below i will demonstrate how the one-to-many relation between role and user is established 

```Python
class User(UserMixin, db.Model):
    __tablename__ = 'users'

role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
```
the role.id column is defined as a foreign key and that establishes the relationship between the two tables. The reason for this relationship is because many users can have the same role so instead of creating a seperate table for each user it makes more sense to reference the same one.

### Database Use in View Functions
 A lot of database operations can be directly handled in the view function as shown below.

 ```Python
 @main.route('/edit-profile', methods=['GET', 'POST'])
@login_required
def edit_profile():
    form = EditProfileForm()
    if form.validate_on_submit():
        current_user.name = form.name.data
        current_user.location = form.location.data
        current_user.about_me = form.about_me.data
        db.session.add(current_user._get_current_object())
        db.session.commit()
        flash('Your profile has been updated.')
        return redirect(url_for('.user', username=current_user.username))
    form.name.data = current_user.name
    form.location.data = current_user.location
    form.about_me.data = current_user.about_me
    return render_template('edit_profile.html', form=form)
```
the template for this from also has to be created 

```Python
{% extends "base.html" %}
{% import "bootstrap/wtf.html" as wtf %}


{% block title %}Flasky - Edit Profile{% endblock %}

{% block page_content %}
<div class="page-header">
    <h1>Edit Your Profile</h1>
</div>
<div class="col-md-4">
    {{ wtf.quick_form(form) }}
</div>
{% endblock %}
```
### Database Migrations with Flask-Migrate
Whenever you need to update your database models then your database will need to be updated too. Luckily instead of having to destroy the old tables in order to create the new ones, also losing all the data within them in the process there is a better process called flask-migrate which is an extension of flask this is downloaded using as shown below.

```Pip
$ pip install flask-migrate
```
once flask-migrate is initialized within __init__.py you can change the model classes however you wish, once you have made any changes you must create a migration script which you can do in the terminal window as shown below 

```bash
$ flask db migrate -m "initial migration"
```

When you've done all this you may now update the dtabase using 

```bash
$ flask db upgrade
```

Using the flask db upgrade command updates the database without having to first destroy, lose all data then rebuild it therefore this method is a great alternative 


## 4. Email
Many forms of applications need to send email confirmation when new users register. The best way to do this is with a flask extension called flask-mail which wraps the smptlib and integrates it with Flask. To install this i used the below example.

```bash
$ pip install flask-mail
```
This extension connect to a Simple Mail Transfer Protocol (SMPT) server and passes emails to it for delivery. Below is an example of the config.py file which this smpt is configured in.

```python
class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') 
    MAIL_SERVER = os.environ.get('MAIL_SERVER', 'smtp.office365.com')
    MAIL_PORT = int(os.environ.get('MAIL_PORT', '587'))
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in \
        ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
```

### Integrating Emails with the Application
So we dont have to create a new email for every single new user we can create a function that does this automatically within the email.py file which is shown below.

```python
from threading import Thread
from flask import current_app, render_template
from flask_mail import Message
from . import mail


def send_async_email(app, msg):
    with app.app_context():
        mail.send(msg)


def send_email(to, subject, template, **kwargs):
    app = current_app._get_current_object()
    msg = Message(app.config['FLASKY_MAIL_SUBJECT_PREFIX'] + ' ' + subject,
                    sender=app.config['FLASKY_MAIL_SENDER'], recipients=[to])
    msg.body = render_template(template + '.txt', **kwargs)
    msg.html = render_template(template + '.html', **kwargs)
    thr = Thread(target=send_async_email, args=[app, msg])
    thr.start()
    return thr
```

This function releis on two configuration keys to specify where the emails will be coming from. The send_email() function takes the destination address, subject, template for the email being sent and a list of key word arguments. The keyword arguments passed by the caller are given to the render_template() so that they can be used by the templates that produce the emails

### Sending Asynchronous Email
To avoid delays during request handling the email send function can moved to a background thread which is shown in the example above.

## 5. Large Application Structure
Even though having my whole application in a single file would be possible this would be absolutely awful therefore i structured the source code for my application into multiple files within a repository to make it easier to locate specific parts of the application therefore easier to repair and diagnose.

![large application structure](/app/static/largeapplicationstructure.jpg "Large application structure")

above is an example oif how i layed out the structure of my applications repository. The structure consist of four top level folders 
- The application lives inside a package called app
- The migrations folder, containing the database migration scripts
- Unit tests are wihtin the test folder 
- venv folder contains the Python virtual enviroment as before 

There are also a couple files that are included 
- requirements.txt consisting of all the libraries to run the application 
- config.py which stores all the configuration settings
-flasky.py which includes the applicatoin instance and a few tasks that help manage the application 

### Configuration Options
Applications often need multiple versions of configuration sets a perfect example of this is development, testing and production enviroments. To combat this issue we will instill a heirarchy of configuration settings shown in the example below.

```python
import os
basedir = os.path.abspath(os.path.dirname(__file__))

class Config:
    SECRET_KEY = os.environ.get('SECRET_KEY') or 'thisisnotthekey1970'
    MAIL_SERVER = os.environ.get('MAIL_SERVER', 'smtp.office365.com')
    MAIL_PORT = int(os.environ.get('MAIL_PORT', '587'))
    MAIL_USE_TLS = os.environ.get('MAIL_USE_TLS', 'true').lower() in \
        ['true', 'on', '1']
    MAIL_USERNAME = os.environ.get('MAIL_USERNAME')
    MAIL_PASSWORD = os.environ.get('MAIL_PASSWORD')
    FLASKY_MAIL_SUBJECT_PREFIX = '[Flasky]'
    FLASKY_MAIL_SENDER = 'Flasky Admin <tstu232@outlook.com>'
    FLASKY_ADMIN = os.environ.get('FLASKY_ADMIN')
    SSL_REDIRECT = False
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    SQLALCHEMY_RECORD_QUERIES = True
    FLASKY_POSTS_PER_PAGE = 20
    FLASKY_FOLLOWERS_PER_PAGE = 50
    FLASKY_COMMENTS_PER_PAGE = 30
    FLASKY_SLOW_DB_QUERY_TIME = 0.5

    @staticmethod
    def init_app(app):
        pass

class DevelopmentConfig(Config):
    DEBUG = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('DEV_DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data-dev.sqlite')

class TestingConfig(Config):
    TESTING = True
    SQLALCHEMY_DATABASE_URI = os.environ.get('TEST_DATABASE_URL') or \
        'sqlite://'
    WTF_CSRF_ENABLED = False

class ProductionConfig(Config):
    SQLALCHEMY_DATABASE_URI = os.environ.get('DATABASE_URL') or \
        'sqlite:///' + os.path.join(basedir, 'data.sqlite')


    @classmethod
    def init_app(cls, app):
        Config.init_app(app)

        # email errors to the administrators
        import logging
        from logging.handlers import SMTPHandler
        credentials = None
        secure = None
        if getattr(cls, 'MAIL_USERNAME', None) is not None:
            credentials = (cls.MAIL_USERNAME, cls.MAIL_PASSWORD)
            if  getattr(cls, 'MAIL_USE_TLS', None):
                secure = ()
        mail_handler = SMTPHandler(
            mailhost=(cls.MAIL_SERVER, cls.MAIL_PORT),
            fromaddr=cls.FLASKY_MAIL_SENDER,
            toaddrs=[cls.FLASKY_ADMIN],
            subject=cls.FLASKY_MAIL_SUBJECT_PREFIX + 'Application Error',
            credentials=credentials,
            secure=secure)
        mail_handler.setLevel(logging.ERROR)
        app.logger.addHandler(mail_handler)


class HerokuConfig(ProductionConfig):
    SSL_REDIRECT = True if os.environ.get('DYNO') else False


    @classmethod
    def init_app(cls, app):
        ProductionConfig.init_app(app)


        # handle reverse proxy server headers
        try:
            from werkzeug.middleware.proxy_fix import ProxyFix
        except ImportError:
            from werkzeug.contrib.fixers import ProxyFix
        app.wsgi_app = ProxyFix(app.wsgi_app)


        # log to stderr
        import logging 
        from logging import StreamHandler
        file_handler = StreamHandler()
        file_handler.setLevel(logging.INFO)
        app.logger.addHandler(file_handler)

class DockerConfig(ProductionConfig):
    @classmethod
    def init_app(cls, app):
        ProductionConfig.init_app(app)

        # log to stderr
        import logging
        from logging import StreamHandler
        file_handler = StreamHandler()
        file_handler.setLevel(logging.INFO)
        app.logger.addHandler(file_handler)


config = {
    'development': DevelopmentConfig,
    'testing': TestingConfig,
    'production': ProductionConfig,
    'heroku': HerokuConfig,
    'docker': DockerConfig,

    'default': DevelopmentConfig
}
```

The configuration base class which is located at the top of the file, are to universal configuration setting for all enviroments. The subclasses define settings that are specific for that enviroment.

To make configuration safer most settings can be imported from the as enviroment variables 

### Application Package
This is where all the application code, templates and static files live the folder is called app.

### The use of an Application Factory
Usualy in a single file version of an application it would be completely exceptable to have all the configuration setting created globally but the issue with this is it makes it extremely difficult to make dynamic changes because by the time the script is running the application instance is already running, so it would be too late to make configuration changes. This is even more important when it comes to unit testing because it is sometimes necessary to run the applicatoin under different configuration settings for better test coverage. Below is an example.

```python
from flask import Flask, render_template
from flask_bootstrap import Bootstrap
from flask_mail import Mail
from flask_moment import Moment
from flask_sqlalchemy import SQLAlchemy
from config import config 
from flask_login import LoginManager
from flask_pagedown import PageDown

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()
pagedown = PageDown()

login_manager = LoginManager()
login_manager.login_view ='auth.login'

def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)

    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)
    login_manager.init_app(app)
    pagedown.init_app(app)


    if app.config['SSL_REDIRECT']:
        from flask_sslify import SSLify
        sslify = SSLify(app)


    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)

    from .auth import auth as auth_blueprint 
    app.register_blueprint(auth_blueprint, url_prefix='/auth')

    from .api import api as api_blueprint
    app.register_blueprint(api_blueprint, url_prefix='/api/v1')
    
    return app
```

This constructor imports most of the Flask extensions but without the application instance therefore creating them uninitialized. The create_app() function is the application factory which takes as an argument the name of configuration to use for the application. The configuration settings stored in one of the classes defined in config.py can then be imported using the from_object() method. The configuratoin object is selected from the the config dictionary so once the application is created and configured, the extensions can be initialized by calling on the init_app() on the extensions that were created earlier

The factory function has created the application instance but applications created with the factory function in its current state are incomplete as they are missing routes and cutom error handlers

### Implementing Application Functionality in a Blueprint 
When using an application factory although great for dynamic changes it creates an issue for routes. Usualy in a single file application the application instance exists in the global scope, so this makes it easier for routes to be defined by the app.route decorator. But now that the application is created at runtime the app.route decorator only begins to exist after the create_app() function is invoked, which is too late. The same applies to error handlers.

We can solve this issue by using a method called Blueprints. A Blueprint is similar to an application in the sense that it can also define routes and errorhandlers. The difference is that when the blueprint defines all these routes and errorhandlers they all remain in a dormant state untill the Blueprint is registered with the application. By using the Blueprints in a global scope, the routes an errorhandlers can be defined in the same way as a single file application. Below is an example of a Blueprint.

```python
from flask import Blueprint

main = Blueprint('main', __name__)

from . import views, errors
from ..models import Permission


@main.app_context_processor
def inject_permissions():
    return dict(Permission=Permission)
```

Blueprints work by intantiating an object of class Blueprint. The constructor for this class takes two arguments: the Blueprint name and the module package where the Blueprint is located. In most applications __name__ variable is enough as a second value for the second argument.

The routes and errorhandlers are stored in the same folder as the Blueprint file. By importing these files within the Blueprint it associates them with the Blueprint. It is important to import the modules at the bottom of the Blueprint file to avoid errors due to circular dependencies.

The Blueprint is registered to the application inside the create_app() function as shown below.

```python
def create_app(config_name):
    #...

    from .main import main as main_blueprint
    app.register_blueprint(main_blueprint)

    #...

    return app
```

### Application Script 
The flasky.py module which is located in the top level of the directory is where the application instance is defined. Shown in example below.

```python
import os


COV = None
if os.environ.get('FLASK_COVERAGE'):
    import coverage
    COV = coverage.coverage(branch=True, include='app/*')
    COV.start()

import sys
import click
from flask_migrate import Migrate, upgrade
from app import create_app, db
from app.models import User, Follow, Role, Permission, Post, Comment

app = create_app(os.getenv('FLASK_CONFIG') or 'default')
migrate = Migrate(app, db)

@app.shell_context_processor
def make_shell_context():
    return dict(db=db, User=User, Follow=Follow, Role=Role,
                Permission=Permission, Post=Post, Comment=Comment)
```
The script begins by creating an application. The configuration for this new application is obtained from FLASK_CONFIG, if no configuration is defined then the default configuration will be used.

### Requirements File
In order to keep track of all the libraries that are needed within the application it is good practice to store them all within a requirements.txt folder

### Unit Testing
For testing i used the standard unittest package that comes from the Python standard library. The setUp() and tearDown() methods of the test case class run before and after each test, and any methods that have the name test_ are executed as tests.

When the test runs the setUp() method creates an enviroment as close to that of a running application. Firstly it creates an application which is configured for testing and activates its context. This step ensures that tests have the same access to the current_app like regular requests do. Then it creates a brand-new database using SQLAlchemy specifically for testing purposes, this is done using the create_all() method. Once all testing is done the database and application are torn down using tearDown().


## User Authentication 
Most applications have the goal of providing a personalised user expierience like remembering the users preferences or registration details. The most comon way to authenticate users is through the user providing a name and password and preferably a secret question that only they know the answer to.

The authentication process requires a frankenstein combination of packages which all work harmonesly together.
- Flask-Login
- Werkzeug
- itsdangerous
- Flask-Mail
- Flask-Bootstrap
- Flask-WTF

### Password Security
To ensure the safety of the passwords that all user provide there is a method called password hashing which takes the password given by the user, adds random components to it then applies several one-way cryptographic transformation to it. Once this process is finished you have a hash belonging to a specific user that can used to verify users. Since this hash is whats stored in the database even if it were hacked the hashes would be undoable therefore making it an extremely safe method of password storage.

### Werkzeug Password Hashing
Below is an example of how werzeugs security module hashes passwords.

```python
generate_password_hash(password, method='pbkdf2:sha256', salt_length=8)
```
When this function has run it returns a password hash as a string that can be stored in the database instead of the actual password. 

When the user tries to log in again the check_password_hash(hash, password) method is used which takes the password hash that was previously created and the password the user has just provided to check when hashed if it matches the current hashed password

### Creating an Authentication Blueprint
As good practice i implemented a Blueprint file for all my authenticatoins routes instead of having every single route on the same file this makes it a lot easier to maintain and update in the future. Example below.

```python
from flask import Blueprint 

auth = Blueprint('auth' __name__)

from . import views
```

The app/auth/views.py module imports the Blueprint and the defines all the routes related to authentication using the route decorater. Below is an example of this method.

```python
from flask import render_template, redirect, request, url_for, flash
from flask_login import login_user, logout_user, login_required, current_user
from . import auth
from .. import db
from ..models import User
from ..email import send_email
from .forms import LoginForm, RegistrationForm, ChangePasswordForm, PasswordResetRequestForm, PasswordResetForm, ChangeEmailForm

@auth.before_app_request
def before_request():
    if current_user.is_authenticated:
        current_user.ping()
        if not current_user.confirmed \
                and request.endpoint \
                and request.blueprint != 'auth' \
                and request.endpoint != 'static':
            return redirect(url_for('auth.unconfirmed'))
```

The auth Blueprint needs to be attached to the applicatoin through the create_app() factory function as shown in he example below.

```python 
    from .auth import auth as auth_blueprint 
    app.register_blueprint(auth_blueprint, url_prefix='/auth')
```

### User Authenticatoin with Flask-Login
When a user logs in to the application their state has to be held whilst they navigate multiple pages, Flask-Login is a specialized extension for this process without being tied to a specific authentication mechanism. Pip install below.

```bash
$ pip install flask-login
```
Flask-Login is commonly used with User objects, for them to work together Flask-Login needs the User models to implement a few common properties such as 
- is_authenticated
- is_active
- is_anonymous
- get_id()

There is an easier way to implement all of these methods using the model class Flask-Login provides called UserMixin shown in the example below.

```python
class User(UserMixin, db.Model):
    __tablename__ = 'users'
    id = db.Column(db.Integer, primary_key = True)
    email = db.Column(db.String(64), unique=True, index=True)
    username = db.Column(db.String(64), unique=True, index=True)
    password_hash = db.Column(db.String(128))
    role_id = db.Column(db.Integer, db.ForeignKey('roles.id'))
    confirmed = db.Column(db.Boolean, default=False)
    name = db.Column(db.String(64))
    location = db.Column(db.String(64))
    about_me = db.Column(db.Text())
    member_since = db.Column(db.DateTime(), default=datetime.utcnow)
    last_seen = db.Column(db.DateTime(), default=datetime.utcnow)
    avatar_hash = db.Column(db.String(32))
    posts = db.relationship('Post', backref='author', lazy='dynamic')
    followed = db.relationship('Follow',
                               foreign_keys=[Follow.follower_id],
                               backref=db.backref('follower', lazy='joined'),
                               lazy='dynamic',
                               cascade='all, delete-orphan')
    followers = db.relationship('Follow',
                                foreign_keys=[Follow.followed_id],
                                backref=db.backref('followed', lazy='joined'),
                                lazy='dynamic',
                                cascade='all, delete-orphan')
    comments = db.relationship('Comment', backref='author', lazy='dynamic')
```

Flask-login is initialized in the application factory as shown below.

```python
from flask import Flask, render_template
from flask_bootstrap import Bootstrap
from flask_mail import Mail
from flask_moment import Moment
from flask_sqlalchemy import SQLAlchemy
from config import config 
from flask_login import LoginManager
from flask_pagedown import PageDown

bootstrap = Bootstrap()
mail = Mail()
moment = Moment()
db = SQLAlchemy()
pagedown = PageDown()

login_manager = LoginManager()
login_manager.login_view ='auth.login'

def create_app(config_name):
    app = Flask(__name__)
    app.config.from_object(config[config_name])
    config[config_name].init_app(app)

    bootstrap.init_app(app)
    mail.init_app(app)
    moment.init_app(app)
    db.init_app(app)
    login_manager.init_app(app)
    pagedown.init_app(app)
```

## Deployment using Docker 

There are many ways to deploy an application for example you could use heroku but in my case i decided to go with Docker because of its efficiency and recourcefulness when creating the application. A feature of Docker is its containers and images it creates, containers work as a type of virtual machine that run on top of a kernel of the host operating machine like a clone. These virtual machines are much more lightweight becuase instead of each virtual machine having its own kernel and virtual hardware the virtualization stops at the kernel.

### Instalation of Docker 
Firstly i went on to the Docker website and followed the instructions there to download Docker CE on my development system. After the installation process is complete i was able to access docker from my command terminal as shown below.

```bash
$ docker version
error during connect: This error may indicate that the docker daemon is not running.: Get "http://%2F%2F.%2Fpipe%2Fdocker_engine/v1.24/version": open //./pipe/docker_engine: The system cannot find the file specified.
Client:
 Cloud integration: v1.0.29
 Version:           20.10.17
 API version:       1.41
 Go version:        go1.17.11
 Git commit:        100c701
 Built:             Mon Jun  6 23:09:02 2022
 OS/Arch:           windows/amd64
 Context:           default
 Experimental:      true
```

### How i built a Container Image
The first thing you have to make sure to make before you can use containers is to create an image for the application. An image is essentialy a snapshot of a containers filesystem this is then used as a template when starting new containers. Docker expects the instructions to create an image to be inside of a folder called Dockerfile shown in the example below.

```python
FROM python:3.6-alpine

ENV FLASK_APP flasky.py
ENV FLASK_CONFIG docker

RUN adduser -D flasky
USER flasky

WORKDIR /OneDrive/Documents/flasky/final

COPY requirements requirements
RUN python -m venv venv
RUN venv/bin/pip install -r requirements/docker.txt

COPY app app
COPY migrations migrations
COPY flasky.py config.py boot.sh ./

#runtime configuration
EXPOSE 5000

ENTRYPOINT ["./boot.sh"]
```

The FROM command is required in all Dockerfile files and it is the base of creating an image. In most cases this image is going to be available publicly in Docker Hub this is Dockers contiainer image repository.

The ENV command defines the runtime enviroment variables. This command takes two arguments the variable name and its value. FLASK_APP and FLASK_CONFIG are also included so all the exact same configuration setting are duplicated. Docker will be given its own configuration class as shown below.

```python
class DockerConfig(ProductionConfig):
    @classmethod
    def init_app(cls, app):
        ProductionConfig.init_app(app)

        # log to stderr
        import logging
        from logging import StreamHandler
        file_handler = StreamHandler()
        file_handler.setLevel(logging.INFO)
        app.logger.addHandler(file_handler)
```

The RUN command executes a command in the form of the context of the container image. The first time this is executed a flasky user is automatically created inside the container. The adduser command is part of the Alpine Linux and is available in the base image of the application selected by the FROM command. 

The USER command selects the user which will create the container and also the user for the remaining commands within the Dockerfile. By default Docker uses the root user by default.

The WORKDIR command defines the top level directory where the application is going to be installed.

The COPY command copies all the files in the local system and pastes them onto the container

The last RUN commands create the virtual enviroment and install all the requirements into it. This comes from the dedicated requirements.txt file that was created.

The EXPOSE command defines which port the application will use 

And the final command ENTRYPOINT will define how the application is executed when the container is initialized. Below is an example of this file.

```python
#!/bin/sh
source venv/bin/activate

while true; do
    flask deploy
    if [[ "$?" == "0" ]]; then
        break
    fi
    echo Deploy command failed, retrying in 5 secs...
    sleep 5
done

exec gunicorn -b :5000 --access-logfile -  --error-logfile - flasky:app
```

When the above scriot starts to run it creates the virtual enviroment first then it runs the applications deploy command thiswill create a new database and insert the default roles. Then a GUNICORN server listening on port 5000 is started. 


### How i built a container image
in the terminal window i typed as shown below.

```bash
$ docker build -t example:latest
```

The -t argument gives a name and tag to the docker container image

When the docker build command successfuly runs the build container image is stored in a local image repository. This can be accessed as shown in the example below.

```bash
$ docker images
REPOSITORY        TAG       IMAGE ID       CREATED       SIZE
final             latest    85aa28c29482   10 days ago   80.8MB
```

### How i ran a container
Once the container image has been created all that you need to do next is run it. This is shown in the example below.

```bash
$ docker run --name examplename -dp 8000:5000 example:latest
```

The --name lets you name the container, this is optional but makes it a lot easier in the future when reviewing containers.

The -dp is a combination -d creating the container in detached mode which allows you to run the application without the dependency of the local system and -p maps port 8000 on the local machine to 5000 inside the container. Below is an example of a container within docker.

```bash
$ docker ps
CONTAINER ID   IMAGE          COMMAND       CREATED       STATUS         PORTS                    NAMES
7a3150c08f40   final:latest   "./boot.sh"   10 days ago   Up 4 seconds   0.0.0.0:8000->5000/tcp   final
```

### Container Orchestration Docker Compose
Containerized applications usually consists of several running containers. As the applicatin grows in complexity it need more containers. Some applications are going to need additional services like message queues or caches. Application that have to handle high loads or need to be fault tolerant will need to scale out by running several instances behinf the load balancer. As the amount of containers that are part of the same application increases it will become more and more difficult to coordinate all them only using Docker. To help with this issue a container orchestration framework is built on top of Docker. Below is an example of one.

```python
version: '3'
services:
  flasky:
    build: .
    ports:
      - "8000:5000"
    env_file: .env
    restart: always
    links:
      - mysql:dbserver
  mysql:
    image: "mysql/mysql-server:5.7"
    env_file: .env-mysql
    restart: always
```

The file shown above is written in YAML, which is simple and implements hierarchial structures that consist of key-value maps and lists.

### What my .env file consists of
When you set the restart key to always it provides a simple way for Docker to automatically restart the container if it exists unexpectidly. Below is an example of my .env file.

```python
FLASK_APP=flasky.py
FLASK_CONFIG=docker
MAIL_USERNAME=**********
MAIL_PASSWORD=**************
```
