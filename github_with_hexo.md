
### 创建仓库

新建一个名为你的用户名.github.io的仓库，比如说，如果你的github用户名是test，那么你就新建test.github.io的仓库（必须是你的用户名，其它名称无效），将来你的网站访问地址就是 http://test.github.io 了


### 绑定域名

域名配置最常见有2种方式，CNAME和A记录，CNAME填写域名，A记录填写IP，由于不带www方式只能采用A记录，所以必须先ping一下你的‘用户名.github.io的IP’，然后到你的域名DNS设置页，将A记录指向你ping出来的IP，将CNAME指向你的‘用户名.github.io’，这样可以保证无论是否添加www都可以访问


### 配置SSH key


### yilia配置gitment

注册github OAuth Application

注意填写：若是绑定个人域名就不能使用yourname.github.io作为Homepage URL和Authorization callback URL。gitment会报Error: Comments Not Initialized错误


