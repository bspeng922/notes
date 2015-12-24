# python paste模块

> python paste是一个WSGI工具包，在WSGI的基础上包装了几层，让应用管理和实现变得方便。

1. python paste.deploy不能只装个paste.deploy包就可以工作了，还需要paste.script包
2. python paste.deploy中loadapp给的路径可用os.path.abspath(配置文件相对路径)得到配置文件的绝对路径，否则报找不到relative_to path
3. python paste.deploy中filter,filter_factory,app,app_factory的规范
    + app是一个callable object，接受的参数(environ,start_response)，这是paste系统交给application的，符合WSGI规范的参数. app需要完成的任务是响应envrion中的请求，准备好响应头和消息体，然后交给start_response处理，并返回响应消息体。
    + filter是一个callable object，其唯一参数是(app)，这是WSGI的application对象，见（1），filter需要完成的工作是将application包装成另一个application（“过滤”），并返回这个包装后的application。
    + app_factory是一个callable object，其接受的参数是一些关于application的配置信息：(global_conf,\*\*kwargs)，global_conf是在ini文件中default section中定义的一系列key-value对，而\*\*kwargs，即一些本地配置，是在ini文件中，app:xxx section中定义的一系列key-value对。app_factory返回值是一个application对象
    + filter_factory是一个callable object，其接受的参数是一系列关于filter的配置信息：(global_conf,\*\*kwargs)，global_conf是在ini文件中default section中定义的一系列key-value对，而\*\*kwargs，即一些本地配置，是在ini文件中，filter:xxx section中定义的一系列key-value对。filter_factory返回一个filter对象

```ini
[DEFAULT]
key1=value1
key2=value2
key3=values

[composite:pdl]
use=egg:Paste#urlmap
/:root
/calc:calc

[pipeline:root]
pipeline = logrequest showversion

[pipeline:calc]
pipeline = logrequest calculator

[filter:logrequest]
username = root
password = root123
paste.filter_factory = pastedeploylab:LogFilter.factory

[app:showversion]
version = 1.0.0
paste.app_factory = pastedeploylab:ShowVersion.factory

[app:calculator]
description = This is an "+-*/" Calculator 
paste.app_factory = pastedeploylab:Calculator.factory
```

```python
import os
import webob
from webob import Request
from webob import Response
from paste.deploy import loadapp
from wsgiref.simple_server import make_server

#Filter
class LogFilter():
    def __init__(self,app):
        self.app = app
        pass
    def __call__(self,environ,start_response):
        print "filter:LogFilter is called."
        return self.app(environ,start_response)
    @classmethod
    def factory(cls, global_conf, **kwargs):
        print "in LogFilter.factory", global_conf, kwargs
        return LogFilter

class ShowVersion():
    def __init__(self):
        pass
    def __call__(self,environ,start_response):
        start_response("200 OK",[("Content-type", "text/plain")])
        return ["Paste Deploy LAB: Version = 1.0.0",]
    @classmethod
    def factory(cls,global_conf,**kwargs):
        print "in ShowVersion.factory", global_conf, kwargs
        return ShowVersion()

class Calculator():
    def __init__(self):
        pass
    
    def __call__(self,environ,start_response):
        req = Request(environ)
        res = Response()
        res.status = "200 OK"
        res.content_type = "text/plain"
        # get operands
        operator = req.GET.get("operator", None)
        operand1 = req.GET.get("operand1", None)
        operand2 = req.GET.get("operand2", None)
        print req.GET
        opnd1 = int(operand1)
        opnd2 = int(operand2)
        if operator == u'plus':
            opnd1 = opnd1 + opnd2
        elif operator == u'minus':
            opnd1 = opnd1 - opnd2
        elif operator == u'star':
            opnd1 = opnd1 * opnd2
        elif operator == u'slash':
            opnd1 = opnd1 / opnd2
        res.body = "%s /nRESULT= %d" % (str(req.GET) , opnd1)
        return res(environ,start_response)
    @classmethod
    def factory(cls,global_conf,**kwargs):
        print "in Calculator.factory", global_conf, kwargs
        return Calculator()
if __name__ == '__main__':
    configfile="pastedeploylab.ini"
    appname="pdl"
    wsgi_app = loadapp("config:%s" % os.path.abspath(configfile), appname)
    server = make_server('localhost',8080,wsgi_app)
    server.serve_forever()
    pass
```


假设在ini文件中, 某条pipeline的顺序是filter1, filter2, filter3, app,那么，最终运行的app_real是这样组织的：
app_real = filter1(filter2(filter3(app)))

在app真正被调用的过程中，filter1.__call__(environ,start_response)被首先调用，若某种检查未通过，filter1做出反应；否则交给filter2__call__(environ,start_response)进一步处理，若某种检查未通过，filter2做出反应，中断链条，否则交给filter3.__call__(environ,start_response)处理，若filter3的某种检查都通过了，最后交给app.__call__(environ,start_response)进行处理。