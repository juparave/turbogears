[Index](index.html)

# Turbogears

## Instalation

### TG 2.4 with Python 3

    $ brew update
    $ brew doctor
    $ brew install python3

    # create Virtual Environment option 1
    $ python3 -m venv tg2.4env

    # create Virtual Environment option 2
    $ virtualenv -p python3 --no-site-packages tg2.4env

    $ source ~/bin/hg-prompt && source ~/workspace/python/{PROJECT_NAME}/tg2.4env/bin/activate

    # --pre latest TG version 
    $ pip install --pre tg.devtools

### Quickstart with Genshi

    $ gearbox quickstart --genshi --skip-default-template PROJECT_NAME

### Flash template

On config/app_cfg.py

Using AppConfig

```python
# my flash template
from string import Template
base_config['flash.default_status'] = 'info'
base_config['flash.template'] = Template(u"""<div class="alert ${container_id} alert-${status} fade in">
                            <button type="button" class="close" data-dismiss="alert">
                              <span aria-hidden="true">×</span>
                              <span class="sr-only">Cerrar</span>
                            </button>
                            ${message}
                        </div>""")
```

Using FullStackApplicationConfigurator

```python
from string import Template
base_config.update_blueprint({
    'flash.default_status': 'info',
    'flash.template': Template(u"""<div class="alert alert-dismissible fade show ${container_id} alert-${status}" role="alert">
                            ${message}
                            <button type="button" class="close" data-dismiss="alert" aria-label="Cerrar">
                              <span aria-hidden="true">&times;</span>
                            </button>
                        </div>""")
})
```

## Deployment Configuration 

### Authorization header missing

ref [Stackoverflow](https://stackoverflow.com/questions/13387516/authorization-header-missing-in-django-rest-framework-is-apache-to-blame)

**Apache mod_wsgi specific configuration**

    # this can go in either server config, virtual host, directory or .htaccess 
    WSGIPassAuthorization On

## Log with full stack

ref [Stackoverflow](https://stackoverflow.com/questions/1508467/log-exception-with-traceback)

```python
import traceback

def gettoken(headers):
    """ Parsing header 'Authorization' value """
    try:
        token = headers.get('Authorization')
        for item in headers.items():
            log.debug("header item: %s", item)
        log.debug("headers: %s", headers)
        log.debug("Authorization: %s", token)
        token = token.replace('Bearer ', '')
    except AttributeError as ex:
        logging.error(u's...\nStack: %s' % (''.join(traceback.format_stack())), exc_info=True)
        token = None
    return token
```

## Generate SSL

### Generate a Key Pair
 
    $ openssl genrsa -out cesarfer.com.key 2048

### Generate the CSR

    $ openssl req -new -key cesarfer.com.key -out cesarfer.com.csr 

**NOTE**: Please do not enter your email address, challenge password or an optional company name when generating the CSR.
[Check CSR](https://ssltools.digicert.com/checker/views/csrCheck.jsp)

## Schema for bootstrap data

To have the chance to erase tables

```python
# -*- coding: utf-8 -*-
"""Setup the [model] application"""
from __future__ import print_function

import sys

from tg import config
import transaction

# to avoid error on python 2
if sys.version_info.major == 3:
    pass
elif sys.version_info.major == 2:
    try:
        input = raw_input
    except NameError:
        pass
else:
    print("Unknown python version - input function not safe")


def setup_schema(command, conf, vars):
    """Place any commands to setup cesarferpromotores here"""
    # Load the models

    # <websetup.websetup.schema.before.model.import>
    from cesarferpromotores import model
    # <websetup.websetup.schema.after.model.import>

    # Destroy previous database
    if 'nose' not in sys.modules:
        confirm = input("Destroy actual database? (yes/no) ")
        if confirm == "yes" or confirm == "":
            print("Destroying previous database")
            model.metadata.drop_all(bind=config['tg.app_globals'].sa_engine)
        else:
            print("Keeping database...")

    # <websetup.websetup.schema.before.metadata.create_all>
    print("Creating tables")
    model.metadata.create_all(bind=config['tg.app_globals'].sa_engine)
    # <websetup.websetup.schema.after.metadata.create_all>
    transaction.commit()
    print('Initializing Migrations')
    import alembic.config
    alembic_cfg = alembic.config.Config()
    alembic_cfg.set_main_option("script_location", "migration")
    alembic_cfg.set_main_option("sqlalchemy.url", config['sqlalchemy.url'])
    import alembic.command
    alembic.command.stamp(alembic_cfg, "head")
```

## Handling project dependencies

It's a good idea to freeze dependencies on development, this way when we deploy the projecto to a production server we won't have problems.

Create requirements.txt

    $ pip freeze > requirements.txt

Find and edit the line of the project package, e.g.:

    ...
    # Editable install with no version control (BikeJack==0.1)
    -e /Users/pablito/workspace/python/BikeJack/tg2.4env/src/bikejack/BikeJack
    ...

Remove it or replace it with repo reference, ref https://pip.pypa.io/en/stable/reference/pip_install/#vcs-support

    ...
    -e "hg+ssh://hg@bitbucket.org/juparave/bikejackserver#egg=bikejack&subdirectory=BikeJack"
    ...


## Troubleshooting

    dyld: Library not loaded: @executable_path/../.Python
    Referenced from: /Library/Frameworks/Python.framework/Versions/3.6/bin/python3.6
    Reason: image not found
    Abort trap: 6

Something happend to python interpreter, maybe after install python3

ref [Matt Andeerson](https://mattbanderson.com/so-you-hosed-your-mac-os-python-install/)
ref [Stackoverflow](https://stackoverflow.com/questions/31768128/pip-installation-usr-local-opt-python-bin-python2-7-bad-interpreter-no-such-f)

Solution 1:

    Only solution in OSX and its variant.

    $ ln -s /usr/local/bin/python /usr/local/opt/python/bin/python2.7
    
Solution 2:

Remove symlinks

    $ find tg2.4env -type l -delete
    
Recreate symlinks

    $ # python 2.7
    $ virtualenv --no-site-packages tg2.4env
    
    $ # python >3
    $ python3 -m venv tg2.4env

### SQLAlchemy requirements and implementation, ZopeTransactionExtension

With SQLAlchemy==1.3.11 and zope.sqlalchemy==1.2 we have to modify `model/__init__.py`

```diff
--- a/webstore/model/__init__.py
+++ b/webstore/model/__init__.py
@@ -1,15 +1,15 @@
 # -*- coding: utf-8 -*-
 """The application's model objects"""

-from zope.sqlalchemy import ZopeTransactionExtension
+from zope.sqlalchemy import register
 from sqlalchemy.orm import scoped_session, sessionmaker
 from sqlalchemy.ext.declarative import declarative_base

 # Global session manager: DBSession() returns the Thread-local
 # session object appropriate for the current web request.
-maker = sessionmaker(autoflush=True, autocommit=False,
-                     extension=ZopeTransactionExtension())
+maker = sessionmaker(autoflush=True, autocommit=False)
 DBSession = scoped_session(maker)
+register(DBSession)
```

### Installing dependencies from requirements file

    $ pip install -r requirements.txt

Install exactly library dependencies and versions

    $ cat requirements.txt | xargs -n 1 pip install
    
To ignore comments (lines starting with a #) and blank lines, use:

    $ cat requirements.txt | cut -f1 -d"#" | sed '/^\s*$/d' | xargs -n 1 pip install
    
### M2Crypto

Dependencies

    $ brew install openssl && brew install swig
    
If already installed, reinstall them, just in case =)

    $ brew reinstall openssl@1.1
    $ brew reinstall swig

Install M2Crypto

    $ brew --prefix openssl
    /usr/local/opt/openssl
    $ LDFLAGS="-L$(brew --prefix openssl)/lib" \
      CFLAGS="-I$(brew --prefix openssl)/include" \
      SWIG_FEATURES="-I$(brew --prefix openssl)/include" \
      pip install m2crypto --no-binary :all:
      
### Marrow Mailer with Python 3

There is an error generated when using Python 3 SMTP library

`ValueError: server_hostname cannot be an empty string or start with a leading dot.`

Monkey-patch this method:

`marrow.mailer.transport.smtp.SMTPTransport.connect_to_server()`

```python
    def connect_to_server(self):
        if self.tls == 'ssl': # pragma: no cover
            connection = SMTP_SSL(host=self.host, local_hostname=self.local_hostname, keyfile=self.keyfile,
                                  certfile=self.certfile, timeout=self.timeout)
        else:
            connection = SMTP(host=self.host, local_hostname=self.local_hostname, timeout=self.timeout)
```

## Handling crawlers

To avoid server overload, limit crawlers access

crawlers_user_agents.py
```python
# -*- coding: utf-8 -*-
from tg import abort

crawlers = ['AdsBot-Google',
            'AppEngine-Google',
            'bitlybot',
            'DF Bot',
            'facebookexternalhit',
            'Go-http-client',
            'Google-Ads-Creatives-Assistant',
            'Google-Ads-Overview',
            'Googlebot',
            'AhrefsBot',
            'Baiduspider',
            'bingbot',
            'coccocbot-web',
            'Dataprovider.com',
            'DotBot',
            'DuckDuckBot',
            'Exabot',
            'MJ12bot',
            'Pinterestbot',
            'SemrushBot',
            'spider',
            'VelenPublicWebCrawler',
            'YandexBot',
            'weborama-fetcher',
            'ZoominfoBot']


def process_request(request):
    try:
        # get first occurrence
        crawler = next((s for s in crawlers if s.lower() in request.user_agent.lower()), None)
    except AttributeError as ex:
        # in case of exception, pass
        crawler = None
    if crawler:
        # HTTPUnauthorized
        abort(401)

```

Inside controllers
```python
from tg.decorators import before_validate

from mastershop.controllers.crawlers_user_agents import process_request

class CartController(BaseController):
    # Uncomment this line if your controller requires an authenticated user
    # allow_only = authorize.not_anonymous()

    def __init__(self, i18n='es'):
        self.i18n = i18n

    @before_validate
    def crawler_blocker(remainder, params):
        """Decorator that blocks certain user agents, mostly web crawlers, search bots"""
        process_request(request)
.
.
.
    @crawler_blocker
    @expose()
    def add(self, pid, **kw):
        """ Add product to cart """
        package = True if 'package' in kw else False
        product = model.Product.getBy_id(pid)
.
.
.

```
