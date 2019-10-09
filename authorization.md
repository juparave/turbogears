## On templates

### **Group** level restriction

```html
 1 <!-- Start: Sidebar Menu -->
 2 <ul class="nav sidebar-menu">
 3    <li class="sidebar-label pt20">Menu</li>
 4    <li py:if="request.identity and ('superadmin' in request.identity['groups'])" class="${('', 'active')[defined('page') and page==page=='account']}">
 5    <a href="${tg.url('/system/account/admin')}">
 6        <span class="fa fa-th-list"></span>
 7        <span class="sidebar-title">Cuentas</span>
 8    </a>
 9    </li>
10 </ul>
```
On line 4, the condition first checks request.identity, if equal to None the user has not been authenticated.  Next, checks if 'superadmin' group is inside the authenticathed user groups by using resquest.identity['groups']

### **Permissions** level restriction

```html
 1 <p>Product name, ${product.name}, <span py:if="request.identity and 'view_price' in request.identity['permissions']"> price ${product.price}</span></p>
```
On line 1, the condition first checks request.identity, if equal to None the user has not been authenticated.  Next, checks if 'view_price' permission is inside the authenticathed user permissions by using resquest.identity['permissions']

## On controllers

```python
 1 # ...
 2 from tg import require
 3 from tg.predicates import Any, is_user, has_permission
 4 # ...
 5 class AdminController(BaseController):
 6     # ...
 7     @expose('funproject.templates.adminlist')
 8     @require(Any(is_user('root'), has_permission('manage'),
 9                  msg='Only administrators can view admin list'))
10     def only_for_admins():
11         flash('Hello admin!')
12         dict()
    # ...
```
On line 8 we specify a require permission to access controller member _only_for_admins_ 


```python
 1 columns.append(ColumnDT('is_serialized', 'is_serialized'))
 2 columns.append(ColumnDT('is_billable', 'is_billable'))
 3 columns.append(ColumnDT('id', 'id'))
 4 if predicates.has_any_permission('view_costs'):
 5     columns.append(ColumnDT('cost_a', 'cost_a', filter=_currency))
 6 else:
 7     columns.append(ColumnDT('zero', 'cost_a', filter=_currency))
```
On line 4 we ask if the user has _view_costs_ permission to view a cost column

