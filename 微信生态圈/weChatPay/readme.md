---
title: 微信生态圈这点事儿(微信H5支付)
date: 2019-10-22
tags: 微信H5支付
---
## 微信H5支付

* 微信内部支付(jsapi支付)

* 微信外部浏览器H5支付(MWEB支付)

**起源:** 为了展示公司价值~~为了让公司生存~~, 为了公司理想和抱负~~为了养活公司上下老小~~, 达成产品变现的伟大前景, 领导决定需要接入H5微信支付。  

**微信支付官方文档**  
[微信支付文档](https://pay.weixin.qq.com/wiki/doc/api/index.html)

### 产品需要做的

1. 设置支付目录
**(请确保实际支付时的请求目录与后台配置的目录一致（现在已经支持配置根目录，配置后有一定的生效时间，一般5分钟内生效），否则将无法成功唤起微信支付。)**  
在微信商户平台（pay.weixin.qq.com）设置您的JSAPI支付支付目录，设置路径：商户平台-->产品中心-->开发配置，如图7.7所示。JSAPI支付在请求支付的时候会校验请求来源是否有在商户平台做了配置，所以必须确保支付目录已经正确的被配置，否则将验证失败，请求支付不成功。
![支付目录](https://pay.weixin.qq.com/wiki/doc/api/img/chapter7_3_1.png)
2. 设置授权域名  
开发JSAPI支付时，在统一下单接口中要求必传用户openid，而获取openid则需要您在公众平台设置获取openid的域名，只有被设置过的域名才是一个有效的获取openid的域名，否则将获取失败。
![公众号设置](https://pay.weixin.qq.com/wiki/doc/api/img/chapter7_3_2.png)
![网页授权域名](https://pay.weixin.qq.com/wiki/doc/api/img/chapter7_3_3.png)

> Talk is cheap. Show me the code.

### 前端需要做(敲)的

**微信登录分为两种:** jsapi支付和微信H5支付

* jsapi支付  
JSAPI支付是用户在微信中打开商户的H5页面，商户在H5页面通过调用微信支付提供的JSAPI接口调起微信支付模块完成支付。应用场景有：
  * 用户在微信公众账号内进入商家公众号，打开某个主页面，完成支付
  * 用户的好友在朋友圈、聊天窗口等分享商家页面连接，用户点击链接打开商家页面，完成支付
  * 将商户页面转换成二维码，用户扫描二维码后在微信浏览器中打开页面后完成支付

* 微信H5支付 
H5支付主要是在手机、ipad等移动设备中通过浏览器来唤起微信支付的支付产品。

### jsapi支付以及微信H5支付

**具体代码:**  
utils-login.js

``` javascript
export const payUtil = {
  /**
   * 当微信js bridge 准备好时  发起微信内部浏览器的微信支付
   * @param {object} data 后端发起订单之后返回的参数
   * @param {function} callback 支付完成后的回调函数
   */
  onBridgeReady (data, callback) {
    WeixinJSBridge.invoke(
      'getBrandWCPayRequest', {
          "appId": webappid,                   // appid
          "timeStamp": data.timeStamp,   // 时间戳
          "nonceStr": data.nonceStr,        // 随机串
          "package": data.package,     
          "signType":"MD5",                    // 微信签名方式：     
          "paySign": data.paySign // 微信签名 
      },
      function (res) {
        if (res.err_msg == "get_brand_wcpay_request:ok") {
          // JS API的返回结果get_brand_wcpay_request:ok仅在用户成功完成支付时返回。
          // 由于前端交互复杂，get_brand_wcpay_request:cancel或者get_brand_wcpay_request:fail可以统一处理为用户遇到错误或者主动放弃，不必细化区分。
          setTimeout(() => {
            // 因为无法判断是否支付完成,所以在此添加一个弹窗,让用户自己手动点击
            mini.showModal({
              title: "提示",
              content: "请确认支付是否已完成?",
              showCancel: true,
              cancelText: "重新支付",
              cancelColor: "#cccccc",
              confirmText: "支付完成",
              confirmColor: "#fd566a",
              success: function (res) {
                if (res.confirm) {
                  if (callback) callback();
                } else {
                  if (callback) callback();
                }
              }
            })
          }, 1000)
        }
      }
    );
  },

  /**
   * 外部js直接发起微信浏览器内进行的微信支付
   * @param {object} data 后端发起订单之后返回的参数
   * @param {function} callback 支付完成后的回调函数
   */
  wechatH5Payment (data, callback) {
    if (typeof WeixinJSBridge == "undefined") {
        if ( document.addEventListener ) {
            document.addEventListener('WeixinJSBridgeReady', this.onBridgeReady(data, callback), false);
        } else if (document.attachEvent) {
            document.attachEvent('WeixinJSBridgeReady', this.onBridgeReady(data, callback)); 
            document.attachEvent('onWeixinJSBridgeReady', this.onBridgeReady(data, callback));
        }
    } else {
        this.onBridgeReady(data, callback);
    }
  },

  /**
   * 外部js直接发起微信外浏览器进行微信支付
   * @param {object} data 后端发起订单之后返回的参数
   */
  wechatWapPayment (data) {
      let href = window.location.href;
      href = encodeURIComponent(addParamsToEnd(href, {key: 'beanTranNo', val: data.beanTranNo}))
      window.location.href = data.mweb_url + '&redirect_url=' + href
  }
}

/**
 * 添加参数到url尾部
 * @param {string} url 需要拼参数的url
 * @param {obj} params 需要拼到末尾的参数  包含key和val
 */
const addParamsToEnd = (url, params) => url.match(/[?]/) ? url + '&' + params.key + '=' + params.val : url + '?' + params.key + '=' + params.val;
```

#### 解析  

* 微信jsapi支付流程  
  1. 调取后端下单接口生成订单
  2. 网页内请求生成支付订单
  3. 后端生成商户订单
  4. 后端调用微信下单api
  5. 微信支付系统生成预付单,并返回给后端预付单信息(prepay_id)
  6. 生成jsapi页面调用的支付参数并签名, 返回支付参数给前端
  7. 用户点击发起支付, jsapi接口请求微信支付系统进行支付
  8. 微信支付系统检查参数是否合法以及授权域名权限, 返回检验结果并请求授权支付
  9. 用户输入密码确认支付, 提交支付授权到微信支付系统, 微信支付系统验证授权
  10. 微信支付系统异步通知后端支付结果
  11. 后端告知微信支付系统处理结果
  12. 前端展示支付消息给用户, 并跳转到指定页面
  13. 前端查询后端支付结果
  14. 后端调用查询api, 查询支付结果, 微信支付系统返回支付结果
  15. 后端返回支付结果到前端, 前端对其进行处理  
**前端主要是发起微信支付系统支付以及和后端交互,加密签名等隐私操作需要后端执行**  

* 微信H5支付流程  
  1. 调取后端下单接口生成订单
  2. 后端生成商户订单, 返回后端URL
  3. 后端返回给前端URL, 前端URL跳转到指定页面(需要自己拼上回调链接用于接收订单支付结果)
  4. 微信支付后台自己操作一波返回校验结果, 校验成功后deeplink调起微信客户端
  5. 微信客户端内支付完成, 微信支付后台主动去找后端发送支付通知, 后端返回接收结果
  6. 微信客户端支付后, 会再次调起原浏览器, 跳转到目标页面
  7. 前端在目标页面获取到参数(订单号)后, 调取后端接口查询支付结果
  8. 后端判断是否支付成功, 返回前端支付结果
  9. 前端根据不同结果展示不同样式  
**前端主要在这里完成一个跳转打开微信客户端和拼接链接页面,在相关页面进行参数处理的工作,前端需要在回调页面加载后判断url上是否包含指定参数,来进行后续操作,否则无法完成整个闭环**

### 后端需要做的

**后端需要自己看文档**
[微信H5支付文档](https://pay.weixin.qq.com/wiki/doc/api/H5.php?chapter=9_20&index=1)
[微信jsapi支付文档](https://pay.weixin.qq.com/wiki/doc/api/jsapi.php?chapter=9_1)
