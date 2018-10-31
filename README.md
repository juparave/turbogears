# Turbogears

## Deployment Configuration 

### Authorization header missing

**Apache mod_wsgi specific configuration**

    # this can go in either server config, virtual host, directory or .htaccess 
    WSGIPassAuthorization On

