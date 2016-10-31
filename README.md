# wxAuthorization
WeiXin web page authorization process

记录在我们的微信服务号中嵌入im-h5的网页的鉴权的过程。

# 1、微信公众号介绍

## 1.1 微信公众号分类
服务号：给企业和组织提供更强大的业务服务与用户管理能力，帮助企业快速实现全新的公众号服务平台。

订阅号：为媒体和个人提供一种新的信息传播方式，构建与读者之间更好的沟通与管理模式。

企业号：为企业或组织提供移动应用入口，帮助企业建立与员工、上下游供应链及企业应用间的连接。

## 1.2 各类别之间的区别
服务号：主要偏于服务交互（类似银行，114，提供服务查询），认证前后都是每个月可群发4条消息；

订阅号：主要偏于为用户传达资讯（类似报纸杂志），认证前后都是每天只可以群发一条消息；

企业号：主要用于公司内部通讯使用，需要先有成员的通讯信息验证才可以关注成功企业号；

# 2、微信公众平台介绍

## 2.1 平台地址
https://mp.weixin.qq.com/

## 2.2 平台界面预览
![Alt text](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/wx-platform.png "微信公众平台界面")

# 3、微信网页授权

## 3.1 什么是微信网页授权？
要想让用户在微信服务号中访问im-h5系统的页面，必须通过微信网页授权机制，来获取用户基本信息，进而实现业务逻辑。

## 3.2 关于网页授权回调域名的说明
1、在微信公众号请求用户网页授权之前，开发者需要先到公众平台官网中的“开发 - 接口权限 - 网页服务 - 网页帐号 - 网页授权获取用户基本信息”的配置选项中，修改授权回调域名。请注意，这里填写的是域名（是一个字符串），而不是URL，因此请勿加 http:// 等协议头；

![Alt text](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/wx-redirect-domain-name.png "微信公众平台界面")

2、授权回调域名配置规范为全域名，比如需要网页授权的域名为：www.qq.com ， 配置以后此域名下面的页面 http://www.qq.com/music.html 、 http://www.qq.com/login.html 都可以进行OAuth2.0鉴权。但 http://pay.qq.com 、 http://music.qq.com 、 http://qq.com 无法进行OAuth2.0鉴权。

3、如果公众号登录授权给了第三方开发者来进行管理，则不必做任何设置，由第三方代替公众号实现网页授权即可 

## 3.3 关于网页授权的两种scope的区别说明
1、服务号获得高级接口后，默认拥有scope参数（授权作用域权限）中的snsapi_base和snsapi_userinfo .

2、以snsapi_base为scope发起的网页授权，是用来获取进入页面的用户的openid的，并且是静默授权并自动跳转到回调页的。用户感知的就是直接进入了回调页（往往是业务页面）。

3、以snsapi_userinfo为scope发起的网页授权，是用来获取用户的基本信息的。但这种授权需要用户手动同意，并且由于用户同意过，所以无须关注，就可在授权后获取该用户的基本信息。 

4、用户管理类接口中的“获取用户基本信息接口”，是在用户和公众号产生消息交互或关注后事件推送后，才能根据用户OpenID来获取用户基本信息。这个接口，包括其他微信接口，都是需要该用户（即openid）关注了公众号后，才能调用成功的。

## 3.4 关于网页授权access_token和普通access_token的区别
1、微信网页授权是通过OAuth2.0机制实现的，在用户授权给公众号后，公众号可以获取到一个网页授权特有的接口调用凭证（网页授权access_token），通过网页授权access_token可以进行授权后接口调用，如获取用户基本信息； 

2、其他微信接口，需要通过基础支持中的“获取access_token”接口来获取到的普通access_token调用。 


## 3.5 关于UnionID机制
1、请注意，网页授权获取用户基本信息也遵循UnionID机制。即如果开发者有在多个公众号，或在公众号、移动应用之间统一用户帐号的需求，需要前往微信开放平台（open.weixin.qq.com）绑定公众号后，才可利用UnionID机制来满足上述需求。 

2、UnionID机制的作用说明：如果开发者拥有多个移动应用、网站应用和公众帐号，可通过获取用户基本信息中的unionid来区分用户的唯一性，因为同一用户，对同一个微信开放平台下的不同应用（移动应用、网站应用和公众帐号），unionid是相同的。


## 3.6 关于特殊场景下的静默授权
1、上面已经提到，对于以snsapi_base为scope的网页授权，就静默授权的，用户无感知； 

2、对于已关注公众号的用户，如果用户从公众号的会话或者自定义菜单进入本公众号的网页授权页，即使是scope为snsapi_userinfo，也是静默授权，用户无感知。


## 3.7 网页授权流程
**第一步：用户同意授权，获取code；**

在确保微信公众账号拥有授权作用域（scope参数）的权限的前提下，引导关注者打开如下页面：

```
https://open.weixin.qq.com/connect/oauth2/authorize?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=SCOPE&state=STATE#wechat_redirect
```

参数说明：

参数 | 是否必须 | 说明 		
-----|------|----
appid   | 是    | 公众号的唯一标识
redirect_uri    | 是    | 授权后重定向的回调链接地址，请使用urlencode对链接进行处理
response_type    | 是    | 返回类型，请填写code
scope    | 是    | 应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且，即使在未关注的情况下，只要用户授权，也能获取其信息）
state   | 否    | 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
#wechat_redirect    | 是    | 无论直接打开还是做页面302重定向时候，必须带此参数

当scope等于snsapi_userinfo时，会出现类似下图所示的授权页面：
		
![Alt text](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/snsapi_userinfo.jpg "scope等于snsapi_userinfo时出现的授权页面")

用户同意授权后，页面将跳转至 redirect_uri/?code=CODE&state=STATE
		
    code说明 ： code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。
	
		
		

