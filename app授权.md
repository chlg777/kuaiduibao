# App授权

## 说明


访问应用级接口时需要进行的授权，比如强制重置用户的密码等。

- url: `/o-auth2/app/access-token`
- method: `POST`
- request:

| 参数名称 | 参数含义 | 数据类型 | 是否可选 | 参数说明 |
| -- | -- | -- | -- | -- |
| app_id | APP ID | int | 必填| 商户应用的唯一标识 |
| app_secret | APP密钥 | String(32) | 必填 | App的密钥 |

- response:

`JSON`

1. 授权成功 （HTTP响应状态码 200）

| 参数名称 | 参数含义 | 数据类型 | 参数说明 |
| -- | -- | -- | -- | -- |
| access_token | 授权令牌 | String(40) | 授权成功后，可进行应用API调用的令牌 |
| expires_in | 有效期 | int | 令牌有效期，单位为秒 |
| token_type | token 类型 | String| 默认 Bearer |
| scope | 授权项 |Boolean | 默认 true |

注： 请对授权令牌进行缓存处理；

2. 失败 （HTTP 响应状态码 非200）

| 参数名称 | 参数含义 | 数据类型 | 参数说明 |
| -- | -- | -- | -- | -- |
| error | 错误名称 | String | 代表错误的范畴 |
| error_description | 错误描述 | String | 说明具体的错误项 |




