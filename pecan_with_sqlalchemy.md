# pecan model with sqlalchemy

官方文档：[Working with Databases, Transactions, and ORM’s](http://pecan.readthedocs.io/en/latest/databases.html)


+ **第一步就是要修改init_model方法**

在model目录的__init__.py里面默认有一个init_model方法，可以用来初始化数据库连接等操作

> The purpose of this method is to determine bindings from your configuration file and create necessary engines, pools, etc. according to your ORM or database toolkit of choice.

```python
from pecan import conf
from sqlalchemy import create_engine, MetaData
from sqlalchemy.orm import scoped_session, sessionmaker

Session = scoped_session(sessionmaker())
metadata = MetaData()


def _engine_from_config(configuration):
    configuration = dict(configuration)
    url = configuration.pop('url')
    return create_engine(url, **configuration)


def init_model():
    conf.sqlalchemy.engine = _engine_from_config(conf.sqlalchemy)


def start():
    Session.bind = conf.sqlalchemy.engine
    metadata.bind = Session.bind


def start_read_only():
    Session.bind = conf.sqlalchemy.engine
    metadata.bind = Session.bind


def commit():
    Session.commit()


def rollback():
    Session.rollback()


def clear():
    Session.remove()

```


+ **第二步就是修改app.py文件，加上hooks**
需要注意的是对于GET、HEAD请求，pecan会调用start_read_only()方法。

```python
from pecan import make_app
from pecan.hooks import TransactionHook
from pecan_model2 import model


def setup_app(config):

    model.init_model()
    app_conf = dict(config.app)

    return make_app(
        app_conf.pop('root'),
        logging=getattr(config, 'logging', {}),
        hooks=[
            TransactionHook(
                model.start,
                model.start_read_only,
                model.commit,
                model.rollback,
                model.clear
            )],
        **app_conf
    )

```

> 
> on HTTP POST, PUT, and DELETE requests, TransactionHook takes care of the transaction automatically by following these rules
> 
> 1. Before controller routing has been determined, model.start() is called. This function should bind to the appropriate SQLAlchemy engine and start a transaction.
> 
> 2. Controller code is run and returns.
> 
> 3. If your controller or template rendering fails and raises an exception, model.rollback() is called and the original exception is re-raised. This allows you to rollback your database transaction to avoid committing work when exceptions occur in your application code.
> 
> 4. If the controller returns successfully, model.commit() and model.clear() are called.
> 
> 
> On idempotent operations (like HTTP GET and HEAD requests), TransactionHook handles transactions following different rules.
> 
> 1. model.start_read_only() is called. This function should bind to your SQLAlchemy engine.
> 
> 2. Controller code is run and returns.
> 
> 3. If the controller returns successfully, model.clear() is called.
> 
 

+ **然后我在model/user.py里面创建了一个简单的User**

```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext import declarative


Base = declarative.declarative_base()


class User(Base):
    __tablename__ = 'users'

    id = Column(Integer, primary_key=True)
    name = Column(String(64), nullable=False)
    password = Column(String(64), nullable=False)
    email = Column(String(64))

```


+ **最后就可以在controller里面调用**

```python
import pecan
from pecan import conf
from pecan import expose, redirect
from webob.exc import status_map

from pecan_model2.model import Session, MetaData
from pecan_model2.model.user import User


class RootController(object):
    @expose(generic=True, template='json')
    def index(self):
        query = Session.query(User)
        users = query.all()
        names = [user.name for user in users]
        return {"names": names}

    @expose('json')
    @index.when(method='POST')
    def index_post(self):
        username = pecan.request.POST.get('username')
        password = pecan.request.POST.get('password')
        email = pecan.request.POST.get('email')

        user = User()
        user.name = username
        user.password = password
        if email:
            user.email = email
        Session.add(user)
        return {"message": "OKKKKK"}
```


+ **这样就可以在页面上调用请求**
   http://127.0.0.1:8080

```
页面返回数据
{
  "users": [
    "admin",
    "admin2",
    "admin3"
  ]
}
```

完整代码 [https://github.com/bspeng922/pecan_model2/](https://github.com/bspeng922/pecan_model2/)