# Turbogears

## Deployment Configuration 

### Authorization header missing

ref [Stackoverflow](https://stackoverflow.com/questions/13387516/authorization-header-missing-in-django-rest-framework-is-apache-to-blame)

**Apache mod_wsgi specific configuration**

    # this can go in either server config, virtual host, directory or .htaccess 
    WSGIPassAuthorization On

