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

![large application structure](flasky/Final/app/static/ large_application_structure.png)





