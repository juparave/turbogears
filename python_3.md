# Python 3 notes

## Dependencies on host computer

For Ubuntu 18.04

    $ sudo apt-get install gcc libpq-dev -y
    $ sudo apt-get install python-dev  python-pip -y
    $ sudo apt-get install python3-dev python3-pip python3-venv python3-wheel -y
    $ pip3 install wheel

## Create environment

Create an evironment inside root repository directory, e.g. `workspace/pyth9on/Chofero`

    $ python3 -m venv tg2.4.3env
    $ source tg2.3.4env/bin/activate
    $ pip install --upgrade pip
    $ pip install tg.devtools
    $ cd Chofero
    $ pip install -e .
    

## __str__ and __repr__

You can see `__str__` as a method for users and `__repr__` as a method for developers.

`__repr__`: The “official” string representation of an object. This is how you would make an object of the class. The goal of __repr__ is to be unambiguous.

`__str__`: The “informal” or nicely printable string representation of an object. This is for the enduser.

```python
class Notification(DeclarativeBase):
    __tablename__ = 'notification'
    __table_args__ = {'mysql_engine': 'MyISAM', 'mysql_charset': 'utf8'}

    # { Columns

    uid = Column(CHAR(32), primary_key=True, nullable=False, default=uuid_gen)

    user_id = Column(CHAR(32), ForeignKey('ap_user.uid', name="fk_user_uid"))

    title = Column(Unicode(64), nullable=False)

    content = Column(Unicode(256), nullable=False)

    link = Column(Unicode(128))

    icon = Column(Unicode(64))

    read = Column(Boolean)

    # }

    # Relations {

    user = relationship('User', backref="notifications")

    # { Special methods

    def __repr__(self):
        return 'Notification(uid={}, title={}, content={})'.format(self.uid,
                                                                   repr(self.title),
                                                                   repr(self.content))

    def __str__(self):
        return '{}: {}'.format(self.title, self.content)
```

## Troubleshooting

### ImportError: cannot import name 'Feature' from 'setuptools'

Update version of zope.interface to at least 5.1.0
