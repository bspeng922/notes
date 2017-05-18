# Build oauth2 server


服务器端需要提供两个接口和两个页面：
两个接口，一个是用来获取访问许可的接口，一个是用来获取Access Token 的接口；
然后再提供一个登录页面和一个授权页面。


### 获取访问许可（Authorization Code）接口

客户端从资源拥有者（最终用户）那里请求授权。授权请求直接发送给资源拥有者，如果资源拥有者为该客户端授权， 则给客户端发送一个访问许可(Authorization Code)

传入参数：
appid 应用ID
callback_url 回调URL

返回信息：
未登录、未授权：跳转到登录页面；
已经登录、未授权：跳转到授权页面；
已经登录、已经授权：跳转到传入的callback_url页面，并返回一个访问许可。



### 获取Access Token 接口

客户端向授权服务器出示自己的私有证书和上一步拿到的访问许可，来请求一个访问令牌（AccessToken）；授权服务器验证客户端的私有证书和访问许可的有效性，如果验证有效，则向客户端发送一个访问令牌，访问令牌包括许可的作用域、有效时间和一些其他属性信息

传入参数：
Appid 应用ID，client_id
Appsecret client_secret
callback_url 回调URL
authorization_code 访问许可

返回信息：
Access Token，Access Token 中包含APPID 和用户信息userID 等。获得Access Token 之后，就可以向资源服务器出示Access Token，访问受保护资源。


### 登录页面

这个页面展示给资源拥有者，输入用户名、密码，登录成功之后，如果未授权，则跳转到授权页面；如果已经授权，直接跳转到callback_url 页面，并返回一个访问许可。


### 授权页面

这个页面也是展示给资源拥有者，如果用户已经授权：则直接跳转到callback_url 页面，并返回一个访问许可；如果用户未授权：出现授权页面，授权成功之后，跳到callback_url 页面，并返回一个访问许可。



新建application
http://localhost:8000/o/applications/


获取token
curl -X POST -d "client_id=vGrfaLjQ5UWH328fSV0f4J985E7RTAOJhfmkmwOM&client_seVYa8hD8rfWlgwJVlBImo8BEsHU5YH08POwxSI9Yk9Z6NRr8uKW4Dt69C&grant_type=password&username=moose&password=Meiyou.123" http://localhost:8000/o/token/

刷新token
curl -X POST -d "grant_type=refresh_token&client_id=<your_client_id>&client_secret=<your_client_secret>&refresh_token=<your_refresh_token>" http://localhost:8000/o/token/

