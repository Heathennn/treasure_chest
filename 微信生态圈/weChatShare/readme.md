---
title: 微信生态圈这点事儿(微信H5分享)
date: 2019-10-21
tags: 微信H5分享
---
## 分享H5链接的图形化展示

![blockchain](https://img2018.cnblogs.com/blog/1244003/201811/1244003-20181102101438689-1611226057.png "分享卡片")
这种分享卡片将链接以一种用户一眼就能看懂得样式进行展示, 内部的标题title, 描述describe, 缩略图img均可以通过前端调取微信jsapi完成效果

### 分享H5链接的图形化展示形式
**起源:** 做内容宣发类产品, 用户体验一定要最好, 最期望的效果是用户一看到这种H5分享卡片就有一种想打开的冲动, 以此来吸引用户, 增加流量, 实现产品的~~诱导用户分享~~完美闭环。于是产品经理为了达到这种美好愿景, ~~威胁~~恳求我完成这个~~坑人~~美好需求。

#### 产品需要做的

1. 微信认证过的公众号:必须是经过认证的，没有认证的或者认证过期的都不可以；
2. 经过备案的域名：必须是备案过的，不然是无法使用的；
3. 绑定域名：首先你需要将需要分享的网址的域名绑定到微信公众平台上面，具体操作：先登录微信公众平台进入“公众号设置”的“功能设置”里填写“JS接口安全域名”；
4. 需要在微信后台进入“开发”的IP白名单里添加一个后端服务器的ip，
通过开发者ID及密码调用获取access_token接口时，需要设置访问来源IP为白名单。

#### 前端需要做的

1. 引入jweixin.js
   * npm引入`npm i weixin-js-sdk`
   * 外部js文件引入`<script src="https://res.wx.qq.com/open/js/jweixin-1.0.0.js"></script>`

2. 通过ajax获取参数 :通过后台给你的接口，取到那些必要的参数，然后你需要把当前分享的页面的url传到后台去，必须动态的获取当前页面，而且一定要对url进行编码，要不然会不起作用；

3. 因为重复上线你手机需要清理缓存，这又是一个麻烦事，手机打开：
[debugx5.qq.com](debugx5.qq.com)  
然后滑到底部，有四个清理缓存的选择框，选择清理就好，不会影响别的地方的缓存，无需担心；
4. 前端需要敲代码了  
utils.js

``` javascript
import wx from "weixin-js-sdk";
import Api from "../api/api"; // 后端接口
const appid = "appid"; // 自己的appid

export const wxJssdk = async (title, desc, imgUrl) => {
  let url = window.location.href.split("#")[0]; // 获取锚点之前的链接
  if (url.indexOf("?") >= 0) {
    url = url.split("?")[0]
  }
  
  const links = window.location.href;
  const res = await Api.shareSign({appid, url});
  const signature = res.data.data.signature.toLowerCase();
  
  wx.config({
    debug: false,
    appId: appid,
    jsapi_ticket: res.data.data.jsapi_ticket,
    timestamp: res.data.data.timestamp,
    nonceStr: res.data.data.noncestr,
    signature: signature,
    jsApiList: [
      "onMenuShareTimeline",
      "onMenuShareAppMessage",
      "onMenuShareQQ",
      "onMenuShareWeibo",
      "onMenuShareQZone"
    ]
  });

  wx.ready(function() {
    wx.onMenuShareTimeline({
      title: title, // 分享标题
      desc: desc, // 分享描述
      link: links, // 分享链接
      imgUrl: imgUrl, // 分享图标
      success: function() {},
      cancel: function() {}
    });
    // 微信分享菜单测试
    wx.onMenuShareAppMessage({
      title: title, // 分享标题
      desc: desc, // 分享描述
      link: links, // 分享链接
      imgUrl: imgUrl, // 分享图标
      success: function() {},
      cancel: function() {}
    });
    wx.onMenuShareQQ({
      title: title, // 分享标题
      desc: desc, // 分享描述
      link: links, // 分享链接
      imgUrl: imgUrl, // 分享图标
      success: function() {},
      cancel: function() {}
    });
    wx.onMenuShareWeibo({
      title: title, // 分享标题
      desc: desc, // 分享描述
      link: links, // 分享链接
      imgUrl: imgUrl, // 分享图标
      success: function() {},
      cancel: function() {}
    });
    wx.onMenuShareQZone({
      title: title, // 分享标题
      desc: desc, // 分享描述
      link: links, // 分享链接
      imgUrl: imgUrl, // 分享图标
      success: function() {},
      cancel: function() {}
    });
  });
  wx.error(function(err) {
    console.log(JSON.stringify(err));
  });
}
```

share.vue

``` javascript
wxJssdk(title, describe, picUrl)
```

#### 后端需要做的
告诉后端让后端自己看文档!