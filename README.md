# wechat_oauth2.0
Node.js通过微信网页授权机制获取用户信息

# 安装
需要安装一个组件：

> npm install
> npm install request --save


# 说明
# 0.目标与前置条件

这一节，我将介绍如何使用微信网页授权机制来拉取微信用户的信息。

在进行本节之前，你需要有一个自己的服务器、域名（如果没有，只能对测试号调试，微信规定了正式号必须使用域名）

微信官方文档请 [点击这里](http://mp.weixin.qq.com/wiki/17/c0f37d5704f0b64713d5d2c37b468d75.html)

---

# 1.部署Express
如果不知道如何部署，可参照： [部署Express](http://www.jianshu.com/p/941ea18069e2)

---

# 2.准备工作

## 2.2 什么是订阅号？什么是服务号？

刚开始开发时，总是被这两个概念弄得云里雾里，所以在此之前，有必要弄清楚这两个概念：

>微信公众平台现在已分成订阅公众号和服务公众号两种类型。 　　
**公众平台服务号**：是公众平台的一种帐号类型，旨在为用户提供服务。如：招商银行、中国南方航空。 　　
**公众平台订阅号**：是公众平台的一种帐号类型，为用户提供信息和资讯。如：骑行西藏、央视新闻。 　　

服务号的权限：
+ 1个月(30天)内仅可以发送1条群发消息
+ 发给订阅用户(粉丝)的消息，会显示在对方的聊天列表中
+ 在发送消息给用户时，用户将收到即时的消息提醒
+ 服务号会在订阅用户(粉丝)的通讯录中
+ 可申请自定义菜单

订阅号的权限
+ 每天(24小时内)可以发送1条群发消息
+ 发给订阅用户(粉丝)的消息，将会显示在对方的订阅号文件夹中
+ 在发送消息给订阅用户(粉丝)时，订阅用户不会收到即时消息提醒
+ 在订阅用户(粉丝)的通讯录中，订阅号将被放入订阅号文件夹中
+ 订阅号不支持申请自定义菜单

另外，进入公众平台 -> 设置 -> 帐号信息 -> 类型 -> 升为服务号/订阅号可设置类型；需要注意的是，公众号只有1次机会可以选择成为服务号/订阅号，类型选择之后不可修改，请慎重选择。

---

## 2.1 申请微信开放平台测试号

> **为什么要用测试号？**
因为正式号的接口未开通，如果要开通，必须通过微信认证。
在获得微信认证之前，使用测试号来开发是一个不错的选择。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/82327-4ddd8e702e497edc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

进入[微信开发平台](https://open.weixin.qq.com) 申请一个测试号。申请成功后会打开一个测试号管理页面。
这里的appID和appsecret就是接下来需要用到的两个重要数据。

![测试号管理页面](http://upload-images.jianshu.io/upload_images/82327-f38b430ad8eb3464.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

## 2.2 进入公众号配置

在你绑定了公众号后，可以登录你注册的微信公众平台帐户，在最下面找到 [基本配置] 项，点击进入也可以进行配置：

![公众平台测试帐号](http://upload-images.jianshu.io/upload_images/82327-247cf82b83d0d1de.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

## 2.3 设置授权回调页面域名

在测试号管理页面中，找到“网页服务”->“网页帐号”，点击右边的“修改”，设置你的授权回调页面域名。

![授权回调页面域名](http://upload-images.jianshu.io/upload_images/82327-95aaa31b9087558b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![设置授权回调页面域名](http://upload-images.jianshu.io/upload_images/82327-f497277f8421ee2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

# 3.服务端代码：

在写代码之前，还需要通过npm安装一个组件：

> npm install request --save

安装完毕后，即可开始。

我们在routes文件夹中增加一个路由文件oauth.js：

```javascript
/**
 * @Module   : Wechat oauth Module
 * @Brief	: Process Wechat oauth
 */
var express = require('express');
var router = express.Router();
var request = require('request');

/* 微信登陆 */
var AppID = '<你的测试号或正式号appid>';
var AppSecret = '<你的测试号或正式号appsecret>';
router.get('/wx_login', function(req,res, next){
	//console.log("oauth - login")
	
	// 第一步：用户同意授权，获取code
	var router = 'get_wx_access_token';
	// 这是编码后的地址
	var return_uri = 'http%3A%2F%2Fwww.onelib.biz%2Foauth%2F'+router;  
	var scope = 'snsapi_userinfo';
	
	res.redirect('https://open.weixin.qq.com/connect/oauth2/authorize?appid='+AppID+'&redirect_uri='+return_uri+'&response_type=code&scope='+scope+'&state=STATE#wechat_redirect');
	
});


router.get('/get_wx_access_token', function(req,res, next){
	//console.log("get_wx_access_token")
	//console.log("code_return: "+req.query.code)
	
	// 第二步：通过code换取网页授权access_token
	var code = req.query.code;
	request.get(
		{   
			url:'https://api.weixin.qq.com/sns/oauth2/access_token?appid='+AppID+'&secret='+AppSecret+'&code='+code+'&grant_type=authorization_code',
		},
		function(error, response, body){
			if(response.statusCode == 200){
				
				// 第三步：拉取用户信息(需scope为 snsapi_userinfo)
				//console.log(JSON.parse(body));
				var data = JSON.parse(body);
				var access_token = data.access_token;
				var openid = data.openid;
				
				request.get(
					{
						url:'https://api.weixin.qq.com/sns/userinfo?access_token='+access_token+'&openid='+openid+'&lang=zh_CN',
					},
					function(error, response, body){
						if(response.statusCode == 200){
							
							// 第四步：根据获取的用户信息进行对应操作
							var userinfo = JSON.parse(body);
							//console.log(JSON.parse(body));
							console.log('获取微信信息成功！');
							
							// 小测试，实际应用中，可以由此创建一个帐户
							res.send("\
								<h1>"+userinfo.nickname+" 的个人信息</h1>\
								<p><img src='"+userinfo.headimgurl+"' /></p>\
								<p>"+userinfo.city+"，"+userinfo.province+"，"+userinfo.country+"</p>\
							");
							
						}else{
							console.log(response.statusCode);
						}
					}
				);
			}else{
				console.log(response.statusCode);
			}
		}
	);
});
module.exports = router;
```

最后，在app.js中引入这个路由：

```
...
var oauth = require('./routes/oauth');
...
app.use('/oauth', oauth);
...
```

---

# 4.测试

先关注该测试号，然后在微信中输入网站： www.onelib.biz/oauth/wx_login

![成功获取用户信息](http://upload-images.jianshu.io/upload_images/82327-318930e1f1a9419f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---

> **原创文章，未经许可，请勿转载**
> 作者：Mike的读书季
> 日期：2016.10.08
