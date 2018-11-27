# SQLAlchemy

## Relationship Joins

### Using foreign_keys

```python
class Account(DeclarativeBase):
    __tablename__ = 'account'
    __table_args__ = {'mysql_engine': 'MyISAM', 'mysql_charset': 'utf8'}

    # { Columns

    uid = Column(CHAR(32), primary_key=True, nullable=False, default=uuid_gen)

    owner_id = Column(CHAR(32), ForeignKey('ch_user.uid', name="fk_account_owner"), index=True)

    name = Column(Unicode(32), nullable=False)

    status = Column(Unicode(32))

    database = Column(Unicode(32))

    users_limit = Column(Integer)

    account_type = Column(Integer)

    account_settings = Column(JSONEncodedDict())

    controllers = Column(JSONEncodedDict())

    templates = Column(JSONEncodedDict())

    # late in payments
    is_warned = Column(Boolean, default=False)

    # very late in payments
    is_suspended = Column(Boolean, default=False)

    created = Column(DateTime, nullable=False, default=func.now())

    updated = Column(DateTime, nullable=False, default=func.now(), onupdate=func.now())

    # }

    # { Relations

    owner = relationship('User', foreign_keys='Account.owner_id')

    # }
```

```python
class User(DeclarativeBase):
    """
    User definition.

    This is the user definition used by :mod:`repoze.who`, which requires at
    least the ``user_name`` column.

    """
    __tablename__ = 'ch_user'
    __table_args__ = {'mysql_engine': 'MyISAM', 'mysql_charset': 'utf8'}

    # Columns {

    uid = Column(CHAR(32), primary_key=True, nullable=False, default=uuid_gen)

    account_id = Column(CHAR(32), ForeignKey('account.uid', name="fk_user_account"))

    user_name = Column(Unicode(64), unique=True, nullable=False)

    email_address = Column(Unicode(64), unique=True, nullable=False)

    first_name = Column(Unicode(64))

    last_name = Column(Unicode(64))

    display_name = Column(Unicode(64))

    user_settings = Column(JSONEncodedDict())

    _password = Column('password', Unicode(128))

    gender = Column(Unicode(64))

    dob = Column(DateTime)

    confirmation_code = Column(Unicode(32))

    last_login = Column(DateTime)

    avatar_image_url = Column(Unicode(255))

    mobile_ping = Column(CHAR(32))

    mobile_token = Column(CHAR(32))

    phone_number = Column(Unicode(16))

    mobile_number = Column(Unicode(16))

    extra_number = Column(Unicode(16))

    is_confirmed = Column(Boolean, default=False)

    has_done_setup = Column(Boolean, default=False)

    is_inspector = Column(Boolean, default=False)

    created = Column(DateTime, nullable=False, default=func.now())

    updated = Column(DateTime, nullable=False, default=func.now(), onupdate=func.now())

    # }

    # { Relations

    account = relationship('Account', backref='users', foreign_keys='User.account_id')

    # }

    # { Special Methods

    def __repr__(self):
        return '<User: user_name=%s, email_address=%s, display_name=%s>' % (
            repr(self.user_name),
            repr(self.email_address),
            repr(self.display_name)
        )

    def __unicode__(self):
        return self.display_name or self.user_name

    @property
    def to_json(self):
        return {
            'uid': self.uid,
            'account_id': self.account_id,
            'user_name': self.user_name,
            'email_address': self.email_address,
            'first_name': self.first_name,
            'last_name': self.last_name,
            'display_name': self.display_name,
            'phone_number': self.phone_number,
            'mobile_number': self.mobile_number,
            'extra_number': self.extra_number,
            'user_settings': self.user_settings,
            'gender': self.gender,
            'dob': self.dob,
            'avatar_image_url': self.avatar_image_url,
            'is_inspector': self.is_inspector,
            'inspector': self.inspector.to_json if self.inspector else None,
            'permissions': [p.permission_name for p in self.permissions]
        }

    # }

    # { Properties

    @property
    def permissions(self):
        """Return a set with all permissions granted to the user."""
        perms = set()
        for g in self.groups:
            perms = perms | set(g.permissions)
        return perms

    # }

    # { Helpers

    @classmethod
    def by_email_address(cls, email):
        """Return the user object whose email address is ``email``."""
        return DBSession.query(cls).filter_by(email_address=email).first()

    @classmethod
    def by_user_name(cls, username):
        """Return the user object whose user name is ``username``."""
        return DBSession.query(cls).filter_by(user_name=username).first()

    @classmethod
    def by_id(cls, uid):
        """Return the user object whose user name is ``username``."""
        return DBSession.query(cls).filter_by(uid=uid).first()

    @classmethod
    def by_id_list(cls, id_list):
        """Return objects whose id in ``id_list``."""

        if id_list is None:
            return []
        elif not isinstance(id_list, list):
            """id_list parameter must be a list"""
            if len(id_list.split(',')) > 1:
                id_list = [x.strip() for x in id_list.split(',')]
            else:
                id_list = [id_list]

        return DBSession.query(cls).filter(cls.uid.in_(id_list)).all()

    @classmethod
    def by_email_address_list(cls, email_list):
        """Return objects whose id in ``email_list``."""

        if email_list is None:
            return []
        elif not isinstance(email_list, list):
            """email_list parameter must be a list"""
            if len(email_list.split(',')) > 1:
                email_list = [x.strip() for x in email_list.split(',')]
            else:
                email_list = [email_list]

        return DBSession.query(cls).filter(cls.email_address.in_(email_list)).all()

    @classmethod
    def by_search_in_account(cls, account, q):
        """Return user objects by search `q`"""
        return DBSession.query(cls) \
            .filter(cls.account == account,
                    or_(cls.first_name.like('%%%s%%' % q),
                        cls.last_name.like('%%%s%%' % q),
                        cls.display_name.like('%%%s%%' % q),
                        cls.email_address.like('%%%s%%' % q))
                    ) \
            .all()

    @classmethod
    def get_all(cls):
        """ Return all users """
        return DBSession.query(cls).all()

    @classmethod
    def get_all_not_in_group_list(cls, group_name_list):
        """ Return all objects """
        if group_name_list is None:
            return []
        elif not isinstance(group_name_list, list):
            """group_name_list parameter must be a list"""
            if len(group_name_list.split(',')) > 1:
                group_name_list = [x.strip() for x in group_name_list.split(',')]
            else:
                group_name_list = [group_name_list]

        return DBSession.query(cls).filter(~cls.groups.any(Group.group_name.in_(group_name_list))).all()

    @classmethod
    def _hash_password(cls, password):
        salt = sha256()
        salt.update(os.urandom(60))
        salt = salt.hexdigest()

        hash = sha256()
        # Make sure password is a str because we cannot hash unicode objects
        hash.update((password + salt).encode('utf-8'))
        hash = hash.hexdigest()

        password = salt + hash

        # Make sure the hashed password is a unicode object at the end of the
        # process because SQLAlchemy _wants_ unicode objects for Unicode cols
        password = password.decode('utf-8')

        return password

    def _set_password(self, password):
        """Hash ``password`` on the fly and store its hashed version."""
        self._password = self._hash_password(password)

    def _get_password(self):
        """Return the hashed version of the password."""
        return self._password

    password = synonym('_password', descriptor=property(_get_password,
                                                        _set_password))

    def validate_password(self, password):
        """
        Check the password against existing credentials.

        :param password: the password that was provided by the user to
            try and authenticate. This is the clear text version that we will
            need to match against the hashed one in the database.
        :type password: unicode object.
        :return: Whether the password is valid.
        :rtype: bool

        """
        hash = sha256()
        hash.update((password + self.password[:64]).encode('utf-8'))
        return self.password[64:] == hash.hexdigest()

    # }
```