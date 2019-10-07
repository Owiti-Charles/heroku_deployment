How to Deploy Django Applications on Heroku
===========================================

# Install heroku CLI
[Sign up](https://signup.heroku.com/) to Heroku.

Then install the [Heroku Toolbelt](https://toolbelt.heroku.com/). It is a command line tool to manage your Heroku apps

After installing the Heroku Toolbelt, open a terminal and login to your account:
```bash
$ heroku login
Enter your Heroku credentials.
Email: jakhax@herokudeploying.com
Password (typing will be hidden):
Authentication successful.
```

# Preparing the Application
In this tutorial I will deploy an existing project, [world of comicon](https://github.com/jakhax/comicon-gallery.git).
It's an very simple open-source Django project, a personal gallery for sharing photos of superheros and villains based on category and location.
Its available on [github](https://github.com/jakhax/comicon-gallery.git) so you can actually clone the repository and follow along or try it on your own existing django project.

## Assumptions
* Your familiar with the basics of django e.g concept of apps, settings, urls, basics of databases 
* You have django application that you want to deploy to heroku
* You are familiar with virtual environments - not a must but the knowledge would be a plus
* Your deployment db is postgres
### tested django versions
* django 1.11
* django 2.0.7

We need to add the following to our project, we will cover each of them in detail in the below section

* Add a `Procfile` in the project root;
* Add `requirements.txt` file with all the requirements in the project root;
* Add `Gunicorn` to `requirements.txt`;
* A `runtime.txt` to specify the correct Python version in the project root;
* Configure `whitenoise` to serve static files.

## Procfile
Heroku apps include a `Procfile` that specifies the commands that are executed by the app’s dynos. 

For more information read on the [heroku documentation](https://devcenter.heroku.com/articles/procfile).

Create a file named `Procfile` in the project root with the following content:
```
web: gunicorn your_project_name.wsgi --log-file -
```

## Runtime.txt
This file contains the python version you are using for heroku to use, create `runtime.txt` in your project root and add your python version in the following format
```
python-3.6.4
```
List of [Heroku Python Runtimes](https://devcenter.heroku.com/articles/python-runtimes).

## Whitenoise: Django Static Files settings
Lets first configure static related parameter in `settings.py`
```python 
BASE_DIR = os.path.dirname(os.path.dirname(os.path.abspath(__file__)))

# Static files (CSS, JavaScript, Images)
# https://docs.djangoproject.com/en/1.9/howto/static-files/
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
STATIC_URL = '/static/'

# Extra places for collectstatic to find static files.
STATICFILES_DIRS = (
    os.path.join(BASE_DIR, 'static'),
)
```
Turns out django does not support serving static files in production. However, WhiteNoise project can integrate into your Django application, and was designed with exactly this purpose in mind.

Lets first install Whitenoise   `pip install whitenoise`

**NOTE** Adding whitenoise to wsgi.py has been **deprecated**

Next, install `WhiteNoise` into your Django application. This is done in `settings.py’s middleware section` (at the top):
```python 
MIDDLEWARE_CLASSES = (
    # Simplified static file serving.
    # https://warehouse.python.org/project/whitenoise/
    'whitenoise.middleware.WhiteNoiseMiddleware',
    ...
```
Add the following setting to `settings.py` in the static files section to enable gzip functionality.
```python
# Simplified static file serving.
# https://warehouse.python.org/project/whitenoise/

STATICFILES_STORAGE = 'whitenoise.storage.CompressedManifestStaticFilesStorage'
```
## Requirements.txt
If your under a virtual environment run the command below to generate the requirements.txt file which heroku will use to install python package dependencies 

`pip freeze > requirements.txt`
Make sure you have the following packages if not install the using pip then run the command above again
```
dj-database-url==0.4.2
Django==2.0.2
dj-static==0.0.6
gunicorn==19.7.1
psycopg2-binary==2.7.4
python-decouple==3.1
unicodecsv==0.14.1
whitenoise==3.3.1
Unipath==1.0
```
If you are following along with the World Of Comicon app you should use the provided requirements.txt as you need to install more python packages, for any app just make sure you have the above packages as a plus.

## Optional but very helpfull settings
### python-decouple and dj-database-url
Python Decouple is a must have app if you are developing with Django. It’s important to keep your application credentials like API Keys, Amazon S3, email parameters, database parameters safe, specially if it’s an open source repository. Also no more development_settings.py and production_settings.py, use just one settings.py for your whole project.

Install it via `pip install python-decouple`

dj-database-url is a simple Django utility allows you to utilize the 12factor inspired `DATABASE_URL` environment variable to configure your Django application.

Install it via `pip install dj-database-url`

Firts create a `.env` file and add it to `.gitignore` so you don’t commit any sensitive data to your remote repository.
below is an example of configurations you can add to the `.env` file.

```python
#just an example, dont share your .env settings
SECRET_KEY='342s(s(!hsjd998sde8$=o4$3m!(o+kce2^97kp6#ujhi'
DEBUG=True #set to false in production
DB_NAME='gallery'
DB_USER='user'
DB_PASSWORD='(s(!hsjd998sde8$l263mk@we8$=o4$3m!(o+342s(s(!'
DB_HOST='127.0.0.1'
MODE='dev' #set to 'prod' in production
ALLOWED_HOSTS='.localhost', '.herokuapp.com', '.127.0.0.1'
```

We then edit `settings.py` to enable decouple to use the `.env` configurations.

 ```python
 import os
import dj_database_url
from decouple import config,Csv
MODE=config("MODE", default="dev")
SECRET_KEY = config('SECRET_KEY')
DEBUG = config('DEBUG', default=False, cast=bool)
 # development
if config('MODE')=="dev":
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.postgresql_psycopg2',
            'NAME': config('DB_NAME'),
            'USER': config('DB_USER'),
            'PASSWORD': config('DB_PASSWORD'),
            'HOST': config('DB_HOST'),
            'PORT': '',
        }
        
    }
# production
else:
    DATABASES = {
        'default': dj_database_url.config(
            default=config('DATABASE_URL')
        )
    }

db_from_env = dj_database_url.config(conn_max_age=500)
DATABASES['default'].update(db_from_env)

ALLOWED_HOSTS = config('ALLOWED_HOSTS', cast=Csv())
```

# Lets deploy now
First make sure you are in the root directory of the repository you want to deploy

Next create the heroku app
```bash
heroku create worldofcomicon
```
Create a postgres addon to your heroku app
```bash
heroku addons:create heroku-postgresql:hobby-dev
```
Next we log in to [Heroku dashboard](https://dashboard.heroku.com) to access our app and configure it

<img src="https://imgur.com/rNuHbRE.png" alt="Heroku dashboard" width="600" height="500">

Click on the Settings menu and then on the button Reveal Config Vars:
Next add all the environment vaiables, by default you should have `DATABASE_URI` configuration created after installing postgres to heroku.

Alternatively you can add all your configurations in `.env` file directly to heroku by running the this command.

```bash
heroku config:set $(cat .env | sed '/^$/d; /#[[:print:]]*$/d')
```
Remember to first set `DEBUG` to false and confirm that you have added all the confuguration variables needed.

<img src="https://imgur.com/rFxZbRf.png" alt="Heroku dashboard" width="700" height="500">

### pushing to heroku

confirm that your application is running as expected before pushing, runtime errors will cause deployment to fail so make sure you have no bugs, you have all the following `Procfile`, `requirements.txt` with all required packages and  `runtime.txt` .

```bash
git push heroku master
```
If you did everything correctly then the deployment should be done after a while with an output like this

```
Counting objects: 1201, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (616/616), done.
Writing objects: 100% (1201/1201), 461.28 KiB | 0 bytes/s, done.
Total 1201 (delta 1139), reused 1193 (delta 1131)
remote: Compressing source files... done.
remote: Building source:
remote:
remote: -----> Python app detected
remote: -----> Installing python-3.6.4
remote:      $ pip install -r requirements.txt

...

remote: -----> Launching...
remote:        Released v5
remote:        https://worldofcomicon.herokuapp.com/ deployed to Heroku
remote:
remote: Verifying deploy.... done.
To https://git.heroku.com/worldofcomicon.git
 * [new branch]      master -> master
 ```

 ### Run migrations
 ```bash
 heroku run python manage.py migrate
```

If you instead wish to push your postgres database data to heroku then run
```bash
heroku pg:push mylocaldb DATABASE_URI --app worldofcomicon
```
You can the open the app in your browser [worldofcomicon](https://worldofcomicon.herokuapp.com/)

# Comment
This process was a lot and you can easily mess up as I did, I suggest analyzing the part where you went wrong and going back to read on what you are supposed to do. I also highly recommend going through official documentations about deploying python projects to heroku as you will get a lot information that can help you debug effectively. I will provide some links in the resources section.

Remember heroku does not offer support for media files in the free tier subscription so find some where else to store those e.g Amazon s3.

# Resources
## heroku Docs
* https://devcenter.heroku.com/articles/heroku-postgresql
* https://devcenter.heroku.com/articles/django-assets
* https://devcenter.heroku.com/articles/python-runtimes
* https://devcenter.heroku.com/articles/procfile
* https://devcenter.heroku.com/articles/getting-started-with-python#introduction
* https://devcenter.heroku.com/articles/deploying-python
* https://devcenter.heroku.com/articles/django-app-configuration
* https://devcenter.heroku.com/articles/python-gunicorn

## very helpfull articles
* https://simpleisbetterthancomplex.com/tutorial/2016/08/09/how-to-deploy-django-applications-on-heroku.html
* https://simpleisbetterthancomplex.com/2015/11/26/package-of-the-week-python-decouple.html


