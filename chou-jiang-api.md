# 抽奖API

本接口提供快兑宝活动抽奖逻辑外放，开发者按需定制自己的游戏页面，开发者在快兑宝后台配置活动，并设置相应的奖品。

## 接口

- 接口网关： [网关](https://docs.kuaiduibao.com/jie-kou-wang-guan.html)
- 请求URL： `/service/operator/start-game`
- 请求方式：POST
- 响应格式：JSON

- 请求参数

|参数名称| 参数含义|数据类型|是否可选| 参数说明|
|---| --- | --- | --- | --- |
| access_token | App令牌 | String(40) | 必填 | [App授权](https://docs.kuaiduibao.com/app%E6%8E%88%E6%9D%83.html)获得令牌|
| app_uid | 开发者用户唯一标识 | String(36) | 必填 | 用户标识 |
| activity_id | 快兑宝活动ID | int(10) | 必填 | 在快兑宝活动库可获得 |

* 响应结果

|参数名称| 参数含义|
|---| --- |
| prize | 活动抽奖获得奖品信息 |
| target | 进入快兑宝“我的奖品”页面。如果配置了运营直达位，则默认拼接运营直达位 |

* * prize 属性说明
     
|参数名称| 参数含义|
|---| --- |
| prize_title | 奖品标题|
| image_url | 奖品URL |
| recharge_type | 充值类型 1: 需填写手机号 2、直接领取（中奖即领取）  3、 实物商品领取 |
|product_no | 奖品对应的商品编号 |
| cards | 奖品信息 （券码、券密） |

* * *  cards 元素说明

|参数名称| 参数含义|
|---| --- |
| card_no | 卡号|
| card_pws | 卡密|
| start_time | 启用时间|
| expire_time | 有效期时间 |



     

code: 0 为表示抽奖成功

```
{
    "code": 0,
    "data": {
        "prize": {
            "prize_title": "也买酒138元进口红酒0元购",
            "image_url": "http://image.mmzhuli.com/386d40b4-1609-4df9-9a9f-cdf7b74857d3?hjw=300&hjh=300",
            "recharge_type": 2,
            "cards": [
                {
                    "card_no": "123456",
                    "card_pws": "",
                    "start_time": "01/09/2017",
                    "expire_time": "04/18/2017"
                }
            ]
        },
        "target": "http://hujiaozhuli.m.superlikeapp.com/treasure/join/win-status-coupons?activity_join_id=13_C426JG3YrPb31vC&utm_source=open-1001237"
    },
    "extra": null
}

```

[code 非0 ]
```
{
    "code": 1,
    "msg": "活动无效!",
    "data": null,
    "extra": null
}

```
