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
		
`code说明 ： code作为换取access_token的票据，每次用户授权带上的code将不一样，code只能使用一次，5分钟未被使用自动过期。`
	
**第二步：通过code换取网页授权access_token，同时获取到openid**
	
首先请注意，这里通过code换取的是一个特殊的网页授权access_token,与基础支持中的access_token（该access_token用于调用其他接口）不同。公众号可通过下述接口来获取网页授权access_token。如果网页授权的作用域为snsapi_base，则本步骤中获取到网页授权access_token的同时，也获取到了openid，snsapi_base式的网页授权流程即到此为止。

`尤其注意：由于公众号的secret和获取到的access_token安全级别都非常高，必须只保存在服务器，不允许传给客户端。后续刷新access_token、通过access_token获取用户信息等步骤，也必须从服务器发起。`

请求方法：

获取code后，请求以下链接获取access_token：

`https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code`

**注意：**该请求必须由**服务端**发起

参数说明：

参数 | 是否必须 | 说明 		
-----|------|----
appid | 是 | 公众号的唯一标识
secret | 是 | 公众号的appsecret
code | 是 | 填写第一步获取的code参数
grant_type | 是 | 填写为authorization_code

正确时返回的JSON数据包如下：

```
{ "access_token":"ACCESS_TOKEN",    
 "expires_in":7200,    
 "refresh_token":"REFRESH_TOKEN",    
 "openid":"OPENID",    
 "scope":"SCOPE" } 
```

参数 | 描述
-----|------
access_token | 网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
expires_in | access_token接口调用凭证超时时间，单位（秒）
refresh_token | 用户刷新access_token
openid | 用户唯一标识，请注意，在未关注公众号时，用户访问公众号的网页，也会产生一个用户和公众号唯一的OpenID
scope | 用户授权的作用域，使用逗号（,）分隔

**第三步：刷新access_token（如果需要）**

由于access_token拥有较短的有效期，当access_token超时后，可以使用refresh_token进行刷新，refresh_token有效期为30天，当refresh_token失效之后，需要用户重新授权。

获取第二步的refresh_token后，请求以下链接获取access_token：

`https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN`

参数 | 是否必须 | 说明
-----|------|------
appid | 是 | 公众号的唯一标识
grant_type | 是 | 填写为refresh_token
refresh_token | 是 | 填写通过access_token获取到的refresh_token参数  

正确时返回的JSON数据包如下：

```
{ "access_token":"ACCESS_TOKEN",  
 "expires_in":7200,   
 "refresh_token":"REFRESH_TOKEN",   
 "openid":"OPENID",   
 "scope":"SCOPE" } 
 ```
 
 参数 | 描述
 -----|------
access_token | 网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
expires_in | access_token接口调用凭证超时时间，单位（秒）
refresh_token | 用户刷新access_token
openid | 用户唯一标识
scope | 用户授权的作用域，使用逗号（,）分隔

**第四步：拉取用户信息(需scope为 snsapi_userinfo)**

如果网页授权作用域为snsapi_userinfo，则此时开发者可以通过access_token和openid拉取用户信息了。

请求方法:

`http：GET（请使用https协议） https://api.weixin.qq.com/sns/userinfo?access_token=ACCESS_TOKEN&openid=OPENID&lang=zh_CN `

参数 | 描述
 -----|------
access_token | 网页授权接口调用凭证,注意：此access_token与基础支持的access_token不同
openid | 用户的唯一标识
lang | 返回国家地区语言版本，zh_CN 简体，zh_TW 繁体，en 英语

返回说明

正确时返回的JSON数据包如下：

```
{"openid":" OPENID",  
 " nickname": NICKNAME,   
 "sex":"1",   
 "province":"PROVINCE"   
 "city":"CITY",   
 "country":"COUNTRY",    
 "headimgurl":    "http://wx.qlogo.cn/mmopen/g3MonUZtNHkdmzicIlibx6iaFqAc56vxLSUfpb6n5WKSYVY0ChQKkiaJSgQ1dZuTOgvLLrhJbERQQ
4eMsv84eavHiaiceqxibJxCfHe/46",  
"privilege":[ "PRIVILEGE1" "PRIVILEGE2"     ],    
 "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL" 
} 
```

参数 | 描述
 -----|------
openid | 用户的唯一标识
nickname | 用户昵称
sex | 用户的性别，值为1时是男性，值为2时是女性，值为0时是未知
province | 用户个人资料填写的省份
city | 普通用户个人资料填写的城市
country | 国家，如中国为CN
headimgurl | 用户头像，最后一个数值代表正方形头像大小（有0、46、64、96、132数值可选，0代表640\*640正方形头像），用户没有头像时该项为空。若用户更换头像，原有头像URL将失效。
privilege | 用户特权信息，json 数组，如微信沃卡用户为（chinaunicom）
unionid | 只有在用户将公众号绑定到微信开放平台帐号后，才会出现该字段。

# 4、实际开发过程简述

**第一步，微信菜单中配置网页授权流程中的第一步中的地址，获取code**

在有赞后台配置菜单链接的地址，如下图示：

![Alt text](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/youzan-menu.png "在有赞后台中配置的体检预约菜单的链接地址")

体检预约：

`https://open.weixin.qq.com/connect/oauth2/authorize?appid=XXXXXXXXXXXXX&redirect_uri=http%3A%2F%2Fim-wechat.ikang.com%2F%23%2FthirdPart%2FwechatLoading%3Ffrom%3D6%26isappinstalled%3D0%26targetState%3DpackageList&response_type=code&scope=snsapi_base&state=123#wechat_redirect`

  参数 | 是否必须 | 说明
  -----|------|------
  appid | 是 | 公众号的唯一标识 
  redirect_uri | 是 | 授权后重定向的回调链接地址，请使用urlencode对链接进行处理。编码前：http://im-wechat.ikang.com/#/thirdPart/wechatLoading?from=6&isappinstalled=0&targetState=packageList  编码后：http%3A%2F%2Fim-wechat.ikang.com%2F%23%2FthirdPart%2FwechatLoading%3Ffrom%3D6%26isappinstalled%3D0%26targetState%3DpackageList
  response_type | 是 | 返回类型，请填写code
  scope | 是 | 应用授权作用域，snsapi_base （不弹出授权页面，直接跳转，只能获取用户openid），snsapi_userinfo （弹出授权页面，可通过openid拿到昵称、性别、所在地。并且，即使在未关注的情况下，只要用户授权，也能获取其信息）
  state | 否 | 重定向后会带上state参数，开发者可以填写a-zA-Z0-9的参数值，最多128字节
  #wechat_redirect | 是 | 无论直接打开还是做页面302重定向时候，必须带此参数


  由于是静默授权，不会弹出请求授权的页面，用户无感知，微信会返回一个code，页面将跳转至 redirect_uri/?code=CODE&state=STATE。

  在我们的系统中（以“体检预约”为例），页面将跳转至wechatLoading页面：

 http://im-wechat.ikang.com/#/thirdPart/wechatLoading?from=6&isappinstalled=0&targetState=packageList&code=CODE&state=STATE
 
 注：这里需要说明一下，在2016年11月16日中午12点之前微信回调回来的地址还是上面这个，但是在2016年11月16日中午12点之后，回调回来的地址就是下面这个了：
 
 http://im-wechat.ikang.com/?code=CODE&state=STATE#/thirdPart/wechatLoading/?from=6&isappinstalled=0&targetState=packageList
 
 它把redirect_uri中#之前的地址认为是回调地址，然后在后面加上参数code和state，然后把redirect_uri中#之后的部分放在这个后面，然后再把redirect_uri中我自带的参数from、isappinstalled、targetState再放到这个后面，重定向回来。
 
 这个地址也能跳转到wechatLoading页面。
 

**第二步，在wechatLoading中通过code拿到open_id，传给统一用户端判断该用户是否已经绑定手机号，如果没有，跳转绑手机号页面，绑定手机号之后，为用户自动创建一个账户，为用户自动登录我们的系统平台，如果已经绑定过，直接为用户自动登录我们的系统平台**
 
 在页面重定向到wechatLoading页面之后，需要在这个页面做一些工作。
 
 第一，就是拿到code、from、targetState的值；
 
 第二，调server端的getOpenId的接口，将code传给server端，server端拿到code后，调微信的网页授权流程第二步的接口，拿到该用户针对该服务号的open_id，server端拿到open_id，传给统一用户端，统一用户端拿到open_id后去判断该用户是否已经在我们的系统中创建账户了，如果已经创建了，说明该用户已经在我们的系统中绑定过手机号了，那么统一用户端直接给该用户创建登录的access_token，自动登录，将access_token传给server端，server端将access_token传给前端，前端拿到access_token，直接存到cookies中，然后跳转到相应的页面页面（比如体检预约页面packageList）；如果统一用户端拿到open_id之后，判断该用户还没有在我们的系统中注册过（没有在我们的系统中创建过账户），即没有绑定过手机号，那么统一用户端将只将open_id回传给server端，server端将open_id传给前端，前端只拿到了open_id，没有拿到access_token，说明该用户没有绑定过手机号，于是前端就跳转到绑定手机号的页面，让用户进行绑定手机号的操作，手机号绑定成功之后，统一用户端会根据手机号和open_id为用户创建账户，然后将access_token传给前端，前端拿到access_token，将之存入cookies，然后跳转业务页面（比如跳转体检预约packageList）。
 
 以上这些判断都是在wechatLoading中进行的。如下图所示：

![Alt text](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/wechatLoading-done.png)
 
 

# 5、微信JS-SDK

## 5.1微信JS-SDK概述

1. 微信JS-SDK是微信公众平台面向网页开发者提供的基于微信内的网页开发工具包。

1. 通过使用微信JS-SDK，网页开发者可借助微信高效地使用拍照、选图、语音、位置等手机系统的能力，同时可以直接使用微信分享、扫一扫、卡券、支付等微信特有的能力，为微信用户提供更优质的网页体验。


## 5.2微信JS-SDK使用步骤

### 步骤一：绑定域名 
先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”。

![设置JS接口安全域名](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/JS-API-safety-DomainName.png)

### 步骤二：引入JS文件
在index.html页面引入如下JS文件：http://res.wx.qq.com/open/js/jweixin-1.0.0.js

![引入JS-SDK](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/link-JS-SDK.png)

### 步骤三：通过config接口注入权限验证配置
所有需要使用JS-SDK的页面必须先注入配置信息，否则将无法调用。

```
wx.config({
    debug: true, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
    appId: '', // 必填，公众号的唯一标识
    timestamp: , // 必填，生成签名的时间戳，调用server端接口获得（由server端生成）
    nonceStr: ‘’, // 必填，生成签名的随机串，调用server端接口获得（由server端生成）
    signature: ‘’,// 必填，签名，调用server端接口获得（由server端生成）    
    jsApiList: [] // 必填，需要使用的JS接口列表
});
```

#### 5.2.1 signature签名算法

** 1.由服务端调用接口(http请求方式: GET )： **

```
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET
```

获取access_token（有效期7200秒，开发者必须在自己的服务全局缓存access_token）

** 2.用第一步拿到的access_token 采用http GET方式请求获得jsapi_ticket（有效期7200秒，开发者必须在自己的服务全局缓存jsapi_ticket）： **

```
https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token=ACCESS_TOKEN&type=jsapi
```

成功返回如下JSON：
```
{
"errcode":0,
"errmsg":"ok",
"ticket":"bxLdikRXVbTPdHSM05e5u5sUoXNKd8-41ZO3MhKoyN5OfkWITDGgnr2fwJ0m9E8NYzWKVZvdVtaUgWvsdshFKA",
"expires_in":7200
}
```

获得jsapi_ticket之后，就可以生成JS-SDK权限验证的签名了。

** 签名算法 **
签名生成规则如下：参与签名的字段包括noncestr（随机字符串）, 有效的jsapi_ticket, timestamp（时间戳）, url（当前网页的URL，不包含#及其后面部分） 。对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）后，使用URL键值对的格式（即key1=value1&key2=value2…）拼接成字符串string1。这里需要注意的是所有参数名均为小写字符。对string1作sha1加密，字段名和字段值都采用原始值，不进行URL 转义。


通过config接口注入权限验证配置在index.run.js中执行：
![通过config接口注入权限验证配置](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/config.png)


### 步骤四：通过ready接口处理成功验证

```
wx.ready(function(){
    // config信息验证后会执行ready方法，所有接口调用都必须在config接口获得结果之后，config是一个客户端的异步操作，所以如果需要在页面加载时就调用相关接口，则须把相关接口放在ready函数中调用来确保正确执行。对于用户触发时才调用的接口，则可以直接调用，不需要放在ready函数中。
});
```

### 步骤五：通过error接口处理失败验证

```
wx.error(function(res){
    // config信息验证失败会执行error函数，如签名过期导致验证失败，具体错误信息可以打开config的debug模式查看，也可以在返回的res参数中查看，对于SPA可以在这里更新签名。
});
```

![通过config接口注入权限验证配置](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/config-success-fail.png)


## 5.3微信支付

在JS-SDK引入成功，权限验证配置信息注入成功之后，可通过调用wx对象的chooseWXPay接口发起一个微信支付请求。

```
wx.chooseWXPay({
    timestamp: 0, // 支付签名时间戳，注意微信jssdk中的所有使用timestamp字段均为小写。但最新版的支付后台生成签名使用的timeStamp字段名需大写其中的S字符
    nonceStr: '', // 支付签名随机串，不长于 32 位
    package: '', // 统一支付接口返回的prepay_id参数值，提交格式如：prepay_id=***）
    signType: '', // 签名方式，默认为'SHA1'，使用新版支付需传入'MD5'
    paySign: '', // 支付签名
    success: function (res) {
        // 支付成功后的回调函数
    }
});
```

前5个参数，均来自于调用统一用户端请求支付接口后返回的数据

### 5.3.1微信支付的数据准备

**1、准备好该订单的相关数据data；**

```
data = {
	body: '',  //商品名称
	out_trade_no: '',  //订单号
	total_fee: '',    //订单价格
	openid: '',   //用户的openid
	orderUserId: ''  //用户的userid
};
```

**2、将data转换成Json字符串，再进行base64编码，得到encodeDataJson；**

**3、将encodeDataJson作为data参数的值放入prePayData对象中；**

```
prePayData = {
	appCode: '',
	payType: '',
	terminalType: '',
	data: ''
};
```

**4、调server端接口，将prePayData传给server端，server端返回一个经过加密的数据encData；**

**5、将第2步得到的数据encodeDataJson作为data参数的值、将第4步得到的数据encData作为encData参数的值放入payData对象中；**

```
payData = {
	appCode: '',
	payType: '',
	terminalType: '',
	data: '',
	encData: ''
};

```

**6、调统一用户端接口，将上一步得到的payData数据传给统一用户端，统一用户端将返回调微信支付接口wx.chooseWXPay时所需的数据（timestamp、nonceStr、package、signType、paySign）；**

### 5.3.2微信支付配置-添加支付授权目录

在进行微信支付时，微信要求：所有使用公众号支付方式发起支付请求的链接地址，都必须在支付授权目录之下。我们需要到微信公众平台中添加支付授权目录，否则，在调用微信支付接口时，将报错。如下图所示：

![支付授权目录](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/allow-pay-directory.png)

** 注意： **
1、添加支付授权目录在公众平台微信支付开发配置公众号支付 中；
2、支付授权目录最多只能设置3个；
3、支付授权目录为实际支付页面的上一级目录；
4、IOS系统和Android系统对应的需配置的支付授权目录不一样。IOS系统：调微信接口获取code时设置的重定向的页面（wechatLoading页面）的上一级目录；Android系统：系统中实际支付页面（payStyle）的上一级目录；


# 6、免登录

原系统中，如果在调server端接口时出现session超时或者access_token失效的情况，server端会返回code=2或者code=3；这时会跳转到登录页面，需要重新登录。

在微信中，需要实现免登录，即任何情况下都不能出现登录页面。当调server端接口，server端返回code=2或者code=3时，需要做相关处理。

![code==2时](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/code%20return%202.png)

![code==3时](https://raw.githubusercontent.com/yaoningvital/MarkdownImages/master/images/wx/code%20return%203.png)




