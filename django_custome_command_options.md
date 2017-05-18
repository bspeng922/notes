# Django自定义命令

## 自定义命令

只要在apps模块下建立名字为management目录，再创建commands目录，然后创建以命令命名的文件，在文件内重写BaseCommand的handle方法即可。

+ 目录结构

> myapp/
> |-- management/
> |   |-- commands/
> |       `-- data_import.py
> |   |   `-- __init__.py
> |   `-- __init__.py

+ data_import.py 文件内容

```
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    help = "Print message to the console."

    def handle(self, *args, **options):
        print "Hello, World!"
```

到此便可以使用data_import命令了：

```
python manage.py data_import
```


## 自定义命令参数

要添加自定义参数，可以在 add_arguments 方法中添加，其中第一个参数为命令行输入的参数名， dest参数为options中的key， default=False为该参数的默认值，如果参数不传入的话就默认为这个值。

```
class Command(BaseCommand):
    def add_arguments(self, parser):
        parser.add_argument(
            '--userfile',
            dest='userfile',
            default=None,
            help='Specifies the user file.',
        )
        parser.add_argument(
            '--teamfile',
            dest='teamfile',
            default=None,
            help='Specifies the team file.',
        )

    def handle(self, *args, **options):
        start_time = time.time()
        user_file = options.get("userfile", None)
        team_file = options.get("teamfile", None)
        end_time = time.time()
        print "Done, TimeUsed: ", (end_time - start_time)
```

命令行执行

```
python manage.py data_import --userfile=/home/user.json --teamfile=/home/team.json
```


## Error

+ AppRegistryNotReady("Apps aren't loaded yet.")

If you’re using Django in a plain Python script — rather than a management command — and you rely on the DJANGO_SETTINGS_MODULE environment variable, you must now explicitly initialize Django at the beginning of your script with:

```
import django
django.setup()
```

Otherwise, you will hit an AppRegistryNotReady exception.

*我写的就是management command，不过在使用进程池（Pool）的时候提示 AppRegistryNotReady ，不用进程池的时候不报错。*

参考链接：[http://django.readthedocs.io/en/latest/releases/1.7.html#standalone-scripts](http://django.readthedocs.io/en/latest/releases/1.7.html#standalone-scripts)


