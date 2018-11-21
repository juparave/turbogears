# Turbogears

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
