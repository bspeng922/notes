# Jinja2 笔记

## Install

```
pip install Jinja2
```

Jinja2 2.7版本开始依赖MarkupSafe模块。如果你通过pip或者easy_install安装Jinja2，它将自动为你安装。

```
>>> from jinja2 import Template
>>> template = Template('Hello {{ name }}!')
>>> template.render(name='John Doe')
u'Hello John Doe!'
```


## 基本用法

Jinja2用到一个中心对象，叫做模板Environment。这个类的实例用于存储配置和全局对象，并用于从文件系统或其它位置加载模板。

```
from jinja2 import Environment, PackageLoader
env = Environment(loader=PackageLoader('yourapplication', 'templates'))
template = env.get_template('mytemplate.html')
print template.render(the='variables', go='here')
```

