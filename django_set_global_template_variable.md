# django 设置模板全局变量


### 1. Writing the context processor function

+ *在app目录创建context_processors.py文件，*


```python

def categories(request):
    all_categories = Category.objects.all()

    return {
        'categories': all_categories,
    }

```

+ *将新建的方法加入到 TEMPLATES 的 context_processors 里面*

```python
TEMPLATES = [
    {
        'BACKEND': 'django.template.backends.django.DjangoTemplates',
        'DIRS': [os.path.join(ROOT_PATH, 'templates')],
        'OPTIONS': {
            'context_processors': [
                ... ...
                "yourapp.context_processors.categories"
            ],
        },
    },
]
```



### 2. Write templatetags

+ *创建templatetags*

在templatetags目录里面创建join_event.py  指定模板路径

```python
# join_event.py

from datetime import datetime

from django import template

from ctfchina.models import Event, PlatformSetting

register = template.Library()


@register.inclusion_tag('join_event_nav.html',
                        takes_context=True)
def join_event(context):
    if 'request' not in context:
        return {}

    upcoming_event_list = []
    try:
        upcoming_event_list = Event.objects \
                .filter(begin_datetime__gt=datetime.now()) \
                .order_by('begin_datetime')
    except Exception, e:
        pass
    return {'upcoming_event_list': upcoming_event_list}

```

+ *模板文件*

```html
{% load i18n %}

{% if upcoming_event_list %}
    <li class="menu-item">
        <a data-name="ctfsevent{{ upcoming_event_list.0.id }}join"
                             href="{% url 'ctfs:join_event' upcoming_event_list.0.id %}">
            Join {{ upcoming_event_list.0.name }}
        </a>
    </li>
{% endif %}

```


+ *在模板中引用tag*

```html
{% load join_event %}

{% join_event %}
```


