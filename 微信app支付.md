# 微信APP支付

如果商户采用的是H5接入方式，商户使用自己的微信APP支付，需要做以下配置信息。

- 阅读对象：

客户端开发人员、产品、服务端研发人员；

## ① 商户后台配置微信APP支付参数

先在商户后台 收银台配置微信APP支付参数。

## ② 客户端开发拦截URL

微信APP支付，快抢平台会以打开URL透传微信APP支付需要的参数。

- url

`kuaiqiang://bill/pay/call-open?appid=123&partnerid=123&timestamp=123&noncestr=123&prepayid=123&package=Sign%3DWXPay`

微信APP支付参数：

| 参数名|描述|
| --- | --- |
| paySign | 微信必须|
| appId| 微信appid|
| timeStamp | 时间戳|
| nonceStr | 随机字符串 |
| prepay_id | 预支付交易会话ID|
| package | Sign=WXPay |
| partnerid | 微信支付商户号 |

获取这个参数后，建议客户端开发人员要比对 appId，以免受到服务伪装攻击。

验证数据的合法性之后，拼装微信支付的参数（appId、appSecret、partnerid），通过微信支付SDK的调起微信APP支付。

## ③ 测试支付

配置完成后，开发者就可以测试支付啦！






