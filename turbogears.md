# Turbogears

## Instalation

### TG 2.4 with Python 3

    $ virtualenv -p python3 --no-site-packages tg2.4env
    $ source ~/bin/hg-prompt && source ~/workspace/python/{PROJECT_NAME}/tg2.4env/bin/activate

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
"""Setup the FortanBiz application"""
from __future__ import print_function

import sys
from tg import config
import transaction


def setup_schema(command, conf, vars):
    """Place any commands to setup fortanbiz here"""
    # Load the models

    # <websetup.websetup.schema.before.model.import>
    from fortanbiz import model
    # <websetup.websetup.schema.after.model.import>

    # Destroy previous databases
    if 'nose' not in sys.modules:
>>> python > 3
        confirm = input("Destroy actual database? ([yes]/no) ")
>>> python == 2.7
        confirm = raw_input("Destroy actual database? ([yes]/no) ")
        if confirm == "yes" or confirm == "":
            print("Destroying previous databases")
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