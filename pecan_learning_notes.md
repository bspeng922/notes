# pecan框架学习
## 安装
### 安装虚拟环境
```shell
$ virtualenv pecan-env
$ cd pecan-env
$ source bin/activate
```

### 安装pecan
```shell
$ pip install pecan
```

## 使用
### 创建pecan项目
```shell
$ pecan create test_project
$ cd test_project
```

** 项目的目录结构 **

```shell
$ tree test_project/
test_project/
├── config.py
├── MANIFEST.in
├── public
│   ├── css
│   │   └── style.css
│   └── images
│       └── logo.png
├── setup.cfg
├── setup.py
└── test_project
    ├── app.py
    ├── controllers
    │   ├── __init__.py
    │   ├── __init__.pyc
    │   ├── root.py
    │   └── root.pyc
    ├── __init__.py
    ├── __init__.pyc
    ├── model
    │   ├── __init__.py
    │   └── __init__.pyc
    ├── templates
    │   ├── error.html
    │   ├── index.html
    │   └── layout.html
    └── tests
        ├── config.py
        ├── __init__.py
        ├── test_functional.py
        ├── test_units.py
        └── test_units.pyc
```

** 开启开发者模式 **

```shell
$ python setup.py develop
```

public: 存放静态文件（css、js、images）
controllers: 存放控制文件
templates: 存放模板文件
model: 存放模型文件
tests: 存放测试类

The test_project/app.py file controls how the Pecan application will be created. This file must contain a setup_app() function which returns the WSGI application object. Generally you will not need to modify the app.py file provided by the base application template unless you need to customize your app in a way that cannot be accomplished using config.

### 运行项目
```shell
$ pecan serve config.py
Starting server in PID 000.
serving on 0.0.0.0:8080, view at http://127.0.0.1:8080
```
访问[http://127.0.0.1:8080/](http://127.0.0.1:8080/)即可看到pecan欢迎页面。

### 添加controller
在controllers目录新建books.py文件，在books.py文件中添加BooksController方法
```python
from pecan import expose

class BooksController(object):
    @expose()
    def index(self):
        return "Welcome to book section."

    @expose()
    def bestsellers(self):
        return "We have 5 books in the top 10."
```
在root中添加对BooksController的引用
```python
import books

class RootController(object):
    ......

    books = books.BooksController() # 变量名books对应url路径
```
添加完之后就可以通过页面访问（[http://127.0.0.1:8080/books/bestsellers](http://127.0.0.1:8080/books/bestsellers)），url即为方法名。

> If a method is not decorated with expose(), Pecan will never route a request to it. 

如果需要多级目录，则添加多个controller，并在controller中注册下级方法即可
```python
class BooksController(object):
    @expose()
    def index(self):
        return "Welcome to book section."

class CatalogController(object):
    ......

    books = BooksController()

class RootController(object):
    ......

    catalog = CatalogController()
```

**更改访问路径**
可以通过expose参数来更改路径名, 指定route之后，原方法名不能够再作为路径访问，会提示 *HTTP 404* 
```python
@pecan.expose(route='some-path')
    def some_path(self):
        return dict()
```

也可以调用pecan.route方法更改controller的路径
```python
pecan.route(RootController, 'books-new', BooksController())
```

**指定请求方式**
通过指定method参数来区分请求的方式
```python
class RootController(object):

    # HTTP GET /
    @expose(generic=True, template='json')
    def index(self):
        return dict()

    # HTTP POST /
    @index.when(method='POST', template='json')
    def index_POST(self, **kw):
        uuid = create_something()
        return dict(uuid=uuid)
```

> Sometimes, the standard object-dispatch routing isn’t adequate to properly route a URL to a controller. Pecan provides several ways to short-circuit the object-dispatch system to process URLs with more control, including the special _lookup(), _default(), and _route() methods. Defining these methods on your controller objects provides additional flexibility for processing all or part of a URL.

**定制Response**
```python
    # return json data
    @expose('json')
    def hello(self):
        response.status = 203
        return {'foo': 'bar'}

    # raise HTTP errors
    @expose('json')
    def hello(self):
        abort(404)
    
    # return an explicit response
    @expose()
    def hello(self):
        return Response('Hello, World!', 202)
```

**传入参数**
```python
from pecan import expose

class RootController(object):
    @expose()
    def index(self, arg):
        return arg

    @expose()
    def args(self, *args):
        return ','.join(args)

    @expose()
    def kwargs(self, **kwargs):
        return str(kwargs)
```

```shell
$ curl http://localhost:8080/?arg=foo
foo
$ curl http://localhost:8080/kwargs?a=1&b=2&c=3
{u'a': u'1', u'c': u'3', u'b': u'2'}
$ curl http://localhost:8080/args/one/two/three
one,two,three
```

**文件上传**
```html
<form action="/upload" method="POST" enctype="multipart/form-data">
  <input type="file" name="file" />
  <button type="submit">Upload</button>
</form>
```

```python
from pecan import expose, request

class RootController(object):
    @expose()
    def upload(self):
        assert isinstance(request.POST['file'], cgi.FieldStorage)
        data = request.POST['file'].file.read()
```

### 模板
默认模板是mako，可以更改default_render参数来更改模板
```python
app = {
    'default_renderer' : 'kajiki',
    # ...
}
```

**使用模板**
```python
class MyController(object):
    @expose('path/to/mako/template.html')
    def index(self):
        return dict(message='I am a mako template')
```

> expose() will use the default template engine unless the path is prefixed by another renderer name.
>   @expose('kajiki:path/to/kajiki/template.html')

override_template() allows you to override the template set for a controller method when it is exposed. 
```python
class MyController(object):
    @expose('template_one.html')
    def index(self):
        # ...
        override_template('template_two.html')
        return dict(message='I will now render with template_two.html')
```

render() allows you to manually render output using the Pecan templating framework. 
```python
@expose()
def controller(self):
    return render('my_template.html', dict(message='I am the namespace'))
```
