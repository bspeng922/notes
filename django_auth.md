### Django 权限模块

在谈论django的权限管理之前，有必要先介绍一下django里面的三种角色：
available：指明用户是否被认为活跃的，一旦登录，就会被置于available；
staff：指明用户是否可以登录到后台管理站点：
superuser：超级用户状态，指明该用户缺省拥有所有权限。

#### 权限表
django的权限认证被绑定在 django.contrib.auth 中，当在把django.contrib.auth加入到INSTALLED_APPS中，运行syncdb，就会在数据库中，建立如下几个表：

+ | auth_group |
+ | auth_group_permissions |
+ | auth_permission |
+ | auth_user |
+ | auth_user_groups |
+ | auth_user_user_permissions |

其实django的权限相关都记录在这几张表中，默认在auth_permission中，会为models中的所有表，分别创建add，delete和change三种权限，要为用户或是组赋予权限，可以直接操作auth_group_permissions或是auth_user_user_permissions表

#### 权限控制实现
首先实现最简单的控制，非登录用户禁止访问某一个view。在Django里面，只需要在相应的视图函数前面增加@login_required修饰符即可。针对已登录用户，主要有两种方式来判断是否有某种权限：
**一、通过修饰符**
```python
from django.contrib.auth.decorators import permission_required

@permission_required('myApp.can_do_someting', login_url="/login/")  
def my_view(request):
    .........

#上面第一个参数是权限，格式：app_label.codename，第二个参数是可选的 login_url 参数，这个参数让你指定登录页面的 URL （缺省为 settings.LOGIN_URL ）。

from django.contrib.auth.decorators import user_passes_test

@user_passes_test(lambda u: u.has_perm('myApp.can_do_someting'))
def my_view(request):
   ..........
```

**二、是在view函数中去判断**
```python
def my_view(request):
    if not request.user.has_perm('myApp.can_do_someting'):
         .......
   else:
        .......
```

本人是比较倾向于第二种方式，因为这样可以根据是否有权限，给前端返回不同的自定义内容，而且用第二种方式也可以很方便地运用在template中，给予不同权限的用户显示出不同的前端界面。


#### Django用户、组与权限
User对象具有两个ManyToManyField字段，groups和user_permissions
可以像其它的django Model一样来访问他们：
```python
myuser.groups = [group_list]
myuser.groups.add(group, group, ...)
myuser.groups.remove(group, group, ...)
myuser.groups.clear()
myuser.user_permissions = [permission_list]
myuser.user_permissions.add(permission, permission, ...)
myuser.user_permissions.remove(permission, permission, ...)
myuser.user_permissions.clear()
```

**权限**
权限是作为一个Model存在的，建立一个权限就是创建一个Permission Model的实例
**name**：必需。50个字符或更少，例如，’Can Vote‘
**content_type**：必需，一个对于django_content_type数据库table的引用，table中含有每个应用中的Model的记录。
**codename**：必需，100个字符或更少，例如，'can_vote'。

如果要为某个Model创建权限：
```python
from django.db import models

class Vote(models.Model):
   ...

    class Meta:
        permissions = (("can_vote", "Can Vote"),)
```
如果这个Model在应用foo中，则权限表示为'foo.can_vote',检查某个用户是否具有权限myuser.has_perm('foo.can_vote')

如果已经在 INSTALLED_APPS配置了django.contrib.auth，它会保证为installed applications中的每个Django Model创建3个缺省权限：add, change 和 delete。

这些权限会在你第一次运行 manage.py migrate(1.7之前为syncdb) 时创建。当时所有的models都会建立权限。在这之后创建的新models会在再次运行 manage.py migrate时创建这些默认权限。这些权限与admin管理界面中的创建，删除，修改行为是一一对应的。
