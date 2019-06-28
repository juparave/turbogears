# Deploying Turbogears app using apache mod_wsgi

## .conf file

Used on /etc/apache/sites-available

```conf
# Directory Permissions.
<Directory /home/bikejack/app/BikeJack>
    <IfVersion < 2.3 >
        Order allow,deny
        Allow from all
    </IfVersion>
    <IfVersion >= 2.3>
        Require all granted
    </IfVersion>
</Directory>

#NameVirtualHost *:80
<VirtualHost *:80>
    ServerName     bikejack.net
    ServerAlias    www.bikejack.net

    # RewriteEngine on
    # ReWriteCond %{SERVER_PORT} !^443$
    # RewriteRule ^/(.*) https://bikejack.net/$1 [NC,R=301,L]
    # DocumentRoot /home/bikejack/static/

    Alias /robots.txt /home/bikejack/www/robots.txt
    Alias /static/ /home/bikejack/static
    Alias /uploads /home/bikejack/www/uploads

    <Location "/static*">
        SetHandler None
    </Location>

    DocumentRoot /home/bikejack/www
    WSGIScriptAlias / /home/bikejack/app/BikeJack/apache/bikejack.net.wsgi
    WSGIDaemonProcess bikejack threads=10 processes=3 python-home=/home/bikejack/tg2.4env
    WSGIProcessGroup bikejack
    WSGIPassAuthorization On

    ErrorLog /var/log/apache2/bikejack_error.log
    CustomLog /var/log/apache2/bikejack_access.log combined

</VirtualHost>
```

* important to add *python-home* on *WGSIDaemonProcess*

## .wsgi file

Used on project apache directory. My own practice

```python
import sys

prev_sys_path = list(sys.path)

import site
site.addsitedir('/home/bikejack/tg2.4env/lib/python3.6/site-packages')

new_sys_path = []
for item in list(sys.path):
    if item not in prev_sys_path:
        new_sys_path.append(item)
        sys.path.remove(item)
sys.path[:0] = new_sys_path

import os, sys
sys.path.append('/home/bikejack/app/BikeJack')

os.environ['PYTHON_EGG_CACHE'] = '/home/bikejack/tg2.4env/.python-eggs'

APP_CONFIG = '/home/bikejack/app/BikeJack/bikejack/config/production.ini'

#Setup loggin
import logging.config
logging.config.fileConfig(APP_CONFIG)

# Load application production.ini file.
from paste.deploy import loadapp
application = loadapp('config:%s' % APP_CONFIG)
```

## Troubleshooting

Sometimes is required to install paste and pastedeploy packages

```bash
    $ pip install paste pastedeploy
```
