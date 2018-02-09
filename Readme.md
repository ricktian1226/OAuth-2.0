##简介
OAuth 2.0 定义了四种客户端授权模式：授权码模式（Authorization Code）、简化模式（implicit）、密码模式（Resource Owner Password Credential）、客户端模式（Client Credential）。

##名词定义
(1)**Client** 第三方应用程序，本文又称为客户端；<br>
(2)**Resource Owner** 资源所有者，本文又称为用户；<br>
(3)**User Agent** 用户代理，浏览器或者app；<br>
(4)**Authorization Server** 认证服务器；<br>
(5)**Resource Server** 资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器可以是同一台服务器，也可以是不同服务器；

##授权模式
授权码模式的特点是通过client的后台服务器，与“服务提供商”的认证服务器进行互动。 功能最完整，流程最严密。

![](AuthorizationCode.png)

（A）Resource Owner访问Client，Client将Resource Owner导向Authorization Server；
申请的url包含以下参数：
>response_type：表示授权类型，必选项，此处的值固定为"code"；<br>
>client_id：表示Client的ID，必选项<br>
>redirect_uri：表示重定向URI，可选项<br>
>scope：表示申请的权限范围，可选项<br>
>state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。<br>

例子：

    GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    Host: server.example.com

（B）Resource Owner选择是否给予Client授权；

（C）假设Reource Owner给予授权，Authorazation Server将Reource Owner导向Client事先指定的“重定向URI”（redirection uri），同时附上一个授权码Authorization Code；
请求的url包含以下参数：
>code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。<br>
>state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。<br>

例子：

    HTTP/1.1 302 Found<br>
    Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz<br>

（D）Client收到Authorization Code，附上之前的（redirection uri），向Authorization Server申请token。这一步是在Client的后台服务器上完成的，对Resource Owner不可见；
Client向Authorization Server申请token的url中包含如下参数：
>grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。<br>
>code：表示上一步获得的授权码，必选项<br>
>redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。<br>
>client_id：表示客户端ID，必选项。<br>

例子：

    POST /token HTTP/1.1
	Host: server.example.com
	Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
	Content-Type: application/x-www-form-urlencoded
	
	grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb

（E）Authorization Server核对了Authorization Code和 redirect uri，确认无误后，向Client发送访问令牌（Access_Token）和刷新令牌（Refresh_Token）；
Authorization Server的回复，包含了如下参数：
>access_token：表示访问令r牌，必选项。<br>
>token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。<br>
>expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。<br>
>refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。<br>
>scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。<br>

例子：

	HTTP/1.1 200 OK
	Content-Type: application/json;charset=UTF-8
	Cache-Control: no-store
	Pragma: no-cache
	{
	"access_token":"2YotnFZFEjr1zCsicMWpAA",
	"token_type":"example",
    "expires_in":3600,
    "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
    "example_parameter":"example_value"
    }

从上面代码可以看到，相关参数使用JSON格式发送（Content-Type: application/json）。此外，HTTP头信息中明确指定不得缓存。
##简化模式
简化模式（implicit grant type）不通过Client的服务器，直接在User Agent中向Authoraztion Server申请token，跳过了"Authorization Code"这个步骤，因此得名。所有步骤在User Agent中完成，令牌对Resource Owner是可见的，且Client不需要认证。

![](implicit.png)

（A）Client将Resource Owner导向Authorization Server；
Client发出的请求包含以下参数：
>response_type：表示授权类型，此处的值固定为"token"，必选项；<br>
>client_id：表示Client的ID，必选项；<br>
>redirect_uri：表示重定向的URI，可选项；<br>
>scope：表示权限范围，可选项；<br>
>state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值；<br>

例子：

	GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
    Host: server.example.com

（B）Resource Owner决定是否给于Client授权;

（C）假设Resource Owner给予授权，Authorization Server将Resource Owner导向Client指定的"Redirection URI"，并在URI的Hash部分包含了access_token；
Authorization Server回复Client的请求中包含如下参数：
>access_token：表示访问令牌，必选项；<br>
>token_type：表示令牌类型，该值大小写不敏感，必选项；<br>
>expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间；<br>
>scope：表示权限范围，如果与客户端申请的范围一致，此项可省略；<br>
>state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数；<br>

例子：

	HTTP/1.1 302 Found
	Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA&state=xyz&token_type=example&expires_in=3600
在上面的例子中，认证服务器用HTTP头信息的Location栏，指定浏览器重定向的网址。注意，在这个网址的Hash部分包含了令牌。

（D）User Agent向Resource Server发出请求，其中不包括上一步收到的Hash值；

（E）Resource Server返回一个网页，其中包含的Script可以获取Hash值中的access_token;

（F）User Agent执行上一步获得的Script，提取出access_token；

（G）User Agent将access_token发给Client；

根据上面的D步骤，下一步浏览器会访问Location指定的网址，但是Hash部分不会发送。接下来的E步骤，服务提供商的资源服务器发送过来的代码，会提取出Hash中的令牌。

##密码模式
密码模式（Resource Owner Password Credentials Grant）中，Resource Owner向Client提供自己的用户名和密码。Client使用这些信息，向"Resource Server"索要授权。

在这种模式中，Resource Owner必须把自己的密码给Client，但是Client不得储存密码。这通常用在Resource Owner对Client高度信任的情况下，比如Client是操作系统的一部分，或者由一个著名公司出品。而Authorization Server只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![](Password.png)

（A）Resource Owner向Client提供用户名和密码；
（B）Client将用户名和密码发给Authorization Server；
Client发出的请求包含如下参数：
>grant_type：表示授权类型，此处的值固定为"password"，必选项；<br>
>username：表示用户名，必选项；<br>
>password：表示用户的密码，必选项；<br>
>scope：表示权限范围，可选项；<br>

例子：

	POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=password&username=johndoe&password=A3ddj3w

（C）Authorization Server确认无误后，向Client提供access_token/refresh_token;
Authorization Server向Client发送access_token

例子：

     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
       "example_parameter":"example_value"
     }

整个过程，client不得保存用户名、密码。

##客户端模式

客户端模式（Client Credentials Grant）指Client以自己的名义，而不是以Resource Owner的名义，向"Resource Server"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，Resource Owner直接向Client注册，Client以自己的名义要求"Resource Server"提供服务，其实不存在授权问题。

（A）Client向Authorization Server进行身份认证，并要求一个access_token;
Client发出的请求，包含如下参数：
>grant_type：表示授权类型，此处的值固定为"clientcredentials"，必选项;<br>
>scope：表示权限范围，可选项;<br>

例子：

     POST /token HTTP/1.1
     Host: server.example.com
     Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
     Content-Type: application/x-www-form-urlencoded

     grant_type=client_credentials

（B）Authoraztion Server 确认无误后，向Client提供access_token;

例子：

     HTTP/1.1 200 OK
     Content-Type: application/json;charset=UTF-8
     Cache-Control: no-store
     Pragma: no-cache

     {
       "access_token":"2YotnFZFEjr1zCsicMWpAA",
       "token_type":"example",
       "expires_in":3600,
       "example_parameter":"example_value"
     }




##参照资料
[理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)