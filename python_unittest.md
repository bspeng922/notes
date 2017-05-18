# python 单元测试

Methods whose names start with the string test with one argument (self) in classes derived from unittest.TestCase are test cases. 

创建测试用例通过继承 unittest.TestCase 来实现.

```
import unittest

def fun(x):
    return x + 1

class MyTest(unittest.TestCase):
    def test(self):
        self.assertEqual(fun(3), 4)
```

文档测试

```
def square(x):
    """Squares x.

    >>> square(2)
    4
    >>> square(-2)
    4
    """

    return x * x

if __name__ == '__main__':
    import doctest
    doctest.testmod()
```






