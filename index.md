[Turbogears](turbogears.html)
[SQLAlchemy](sqlalchemy.html)
[Deploying mod_wsgi](deploying_mod_wsgi.html)
[Export data in xlsx format](export_xlsx.html)

## PreMailer

Useful tool to send formated emails

[Github](https://github.com/peterbe/premailer)
[premailer.io](https://premailer.io/)

## iPython

# Using global variables

```python
ipy = get_ipython()
setattr(ipy.__class__, 'user_global_ns', property(lambda self: self.user_ns))
```
