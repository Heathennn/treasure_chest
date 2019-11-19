---
title: 微信生态圈这点事儿(微信H5登录)
date: 2019-10-22
tags: 微信H5登录
---
## 微信H5登录

**起源:** 起源这块不必多说, 微信生态没有登录, 宛若斗鱼没有主播, IG没有the shy。  

### 产品需要做的

1. 登陆公众号，开发 - 接口权限 - 网页服务 - 网页帐号 - 网页授权获取用户基本信息 设置 授权回调域名
   * 注意：这里额外需要传一个txt文件到域名对应的服务器上（微信的安全考虑）
2. 同样的,跟微信H5分享一样需要绑定后端服务器的ip白名单,在微信后台进入“开发”的IP白名单里添加一个后端服务器的ip，
通过开发者ID及密码调用获取access_token接口时，需要设置访问来源IP为白名单。

> Talk is cheap. Show me the code.

### 前端需要做(敲)的

**微信登录分为两种:** 静默登陆和非静默登录

* 静默登陆  
以snsapi_base为scope发起的网页授权，是用来获取进入页面的用户的openid的，并且是静默授权并自动跳转到回调页的。用户感知的就是直接进入了回调页（往往是业务页面）

* 非静默登陆  
以snsapi_userinfo为scope发起的网页授权，是用来获取用户的基本信息的。但这种授权需要用户手动同意，并且由于用户同意过，所以无须关注，就可在授权后获取该用户的基本信息。

**关于特殊场景下的静默授权**

1、上面已经提到，对于以snsapi_base为scope的网页授权，就静默授权的，用户无感知；

2、对于已关注公众号的用户，如果用户从公众号的会话或者自定义菜单进入本公众号的网页授权页，即使是scope为snsapi_userinfo，也是静默授权，用户无感知。

**具体而言，网页授权流程分为四步:**

1、引导用户进入授权页面同意授权，获取code

2、通过code换取网页授权access_token（与基础支持中的access_token不同）

3、如果需要，开发者可以刷新网页授权access_token，避免过期

4、通过网页授权access_token和openid获取用户基本信息（支持UnionID机制）

**具体代码:**  
utils-login.js

``` javascript
export const wxLogin = () => {
  let url = window.location.href;
  let defaultConfig = Object.assign({}, {
    redirect_uri: url,
    scope: "snsapi_userinfo", // 如果静默登陆用snsapi_base
    state: 0,
    response_type: "code",
    appId: webappid
  })

  let wxLoginUrl =
    "https://open.weixin.qq.com/connect/oauth2/authorize?appid=" +
    defaultConfig.appId +
    "&redirect_uri=" +
    encodeURIComponent(defaultConfig.redirect_uri) +
    "&response_type=" +
    defaultConfig.response_type +
    "&scope=" +
    defaultConfig.scope +
    "&state=" +
    defaultConfig.state +
    "#wechat_redirect";

  window.location.href = wxLoginUrl;
}
```

pages  login.vue(login.html)

``` javascript
let code = getQueryString("code");
if (code) {
  let userData = await webLogin({ code }); // 根据code,让后端获取登录信息
}
```

### 后端需要做的
**后端需要自己看文档**
[微信官方文档](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/Wechat_webpage_authorization.html)
