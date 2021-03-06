[Index](index.html)

# SQLAlchemy

#### Connection uri

    connect_string = 'mysql+pymysql://{}:{}@{}:{}/{}?charset=utf8mb4'.format(DB_USER, DB_PASS, DB_HOST, DB_PORT, DATABASE)
    
On table definitions e.g.

```python
class Tags(DeclarativeBase):
    """
    Para relacionar los productos se usarán tags
    """
    __tablename__ = 'tags'
    __table_args__ = {'mysql_engine': 'MyISAM', 'mysql_charset': 'utf8mb4', 'mysql_collate': 'utf8mb4_general_ci'}

    id = Column(u'id', Integer, primary_key=True, nullable=False)

    name = Column(u'name', Unicode(254), index=True)
```



## MySQL v8

### datatables.py

CompileError: Can't resolve label reference for ORDER BY / GROUP BY. Textual SQL expression 'product.catalog' should be explicitly declared as text('product.catalog')

```diff
@@ -364,7 +364,7 @@ class DataTables:
                 else:
                     tablename = self.sqla_object.__table__.name

-            sort_name = "%s.%s" % (tablename, sort_name)
+            sort_name = text("%s.%s" % (tablename, sort_name))
             self.query = self.query.order_by(
                 asc(sort_name) if sort.dir == 'asc' else desc(sort_name))

```

### Incompatible with sql_mode=only_full_group_by

Modify my.cnf with following content

/usr/local/etc/my.cnf

    sql-mode="STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION"

## MySQL and Python 3

Prefered dependency:

```python
install_requires = [
...
    "pymysql"
]
```

Connector
```
sqlalchemy.url = mysql+pymysql://masterits:masteritspass01@localhost/masterits?charset=utf8mb4
```

## Relationship Joins

### Using foreign_keys

```python
class Account(DeclarativeBase):
    __tablename__ = 'account'
    __table_args__ = {'mysql_engine': 'MyISAM', 'mysql_charset': 'utf8mb4', 'mysql_collate': 'utf8mb4_general_ci'}

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
    __table_args__ = {'mysql_engine': 'MyISAM', 'mysql_charset': 'utf8mb4', 'mysql_collate': 'utf8mb4_general_ci'}

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
```

## Sorting ordering

Ordering by created date descending

```python
    @classmethod
    def get_all(cls, inspection_type=None):
        """ Return all objects """

        if inspection_type:
            return DBSession.query(cls).filter(cls.inspection_type == inspection_type).all()

        return DBSession.query(cls).order_by(cls.created.desc()).all()

```

### Using aliased

```python
    # aliases
    assigned_user = aliased(model.User)
    identified_by_user = aliased(model.User)
    # defining columns
    columns = [ColumnDT(model.Issue.uid, 'uid'),
                ColumnDT(model.IssueStatus.name, 'current_status'),
                ColumnDT(identified_by_user.display_name, 'identified_by'),
                ColumnDT(assigned_user.display_name, 'assigned_to'),
                ColumnDT(model.Issue.priority, 'priority'),
                ColumnDT(model.Issue.summary, 'summary'),
                ColumnDT(model.Issue.issue_date, 'issue_date'),
                ColumnDT(model.Issue.target_resolution_date, 'target_resolution_date'),
                ColumnDT(model.Department.name, 'department_name'),
                ColumnDT(model.Department.name, 'department_name'),
                ]

    query = model.DBSession.query(model.Issue)\
        .join(model.Issue.department)\
        .join(identified_by_user, identified_by_user.user_id == model.Issue.identified_by_id, isouter=True)\
        .join(assigned_user, assigned_user.user_id == model.Issue.assigned_to_id, isouter=True)\
        .join(model.Issue.current_status, isouter=True)
```

### Using Text

```python    
    @classmethod
    def get_weekly_stats(cls, user=None):
        """Return Issue stats by issue current status"""

        last_week_query = text("""
        SELECT sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 0 THEN 1 ELSE 0 END) AS `Domingo`,
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 1 THEN 1 ELSE 0 END) AS `Lunes`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 2 THEN 1 ELSE 0 END) AS `Martes`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 3 THEN 1 ELSE 0 END) AS `Miercoles`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 4 THEN 1 ELSE 0 END) AS `Jueves`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 5 THEN 1 ELSE 0 END) AS `Viernes`,
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 0 THEN 1 ELSE 0 END) AS `Sabado`
               from tg_issue 
               WHERE YEARWEEK(tg_issue.issue_date) = YEARWEEK(NOW() - INTERVAL 1 WEEK)""")

        last_week_stats = DBSession.execute(last_week_query).first()

        this_week_query = text("""
        SELECT sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 0 THEN 1 ELSE 0 END) AS `Domingo`,
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 1 THEN 1 ELSE 0 END) AS `Lunes`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 2 THEN 1 ELSE 0 END) AS `Martes`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 3 THEN 1 ELSE 0 END) AS `Miercoles`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 4 THEN 1 ELSE 0 END) AS `Jueves`, 
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 5 THEN 1 ELSE 0 END) AS `Viernes`,
               sum(CASE WHEN WEEKDAY(tg_issue.issue_date) = 0 THEN 1 ELSE 0 END) AS `Sabado` 
               from tg_issue 
               WHERE YEARWEEK(tg_issue.issue_date) = YEARWEEK(NOW())""")

        this_week_stats = DBSession.execute(this_week_query).first()

        stats = {
            'last_week_stats': last_week_stats,
            'this_week_stats': this_week_stats
        }

        return stats
```

Auto calculate ceil

```python
    import math
    max_issues = math.ceil(
        max(
            max(weekly_stats['this_week_stats']),
            max(weekly_stats['last_week_stats']) / 5 )) * 5
```
