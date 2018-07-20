# iOS接入

### 说明

本文档适用于iOS开发人员

### 对接

##### 1 根据需要对接的功能分为必选和非必选

1.1 必选

```
* 基础性功能 ，Native 和 JS的交互 例如:分享的触发，JS使用系统全局粘贴板
```

1.2 非必选 \(根据实际需要\)

```

* 调起客户端登录功能
* 调起自己的客户端支付

```

##### 2 配置浏览器的UserAgent

```
* 必须追加字符串"Openjoy SDK/2.00" 到webview的UA中
```

```c
// 示例代码
NSString *userAgent = [[[UIWebView alloc] init] stringByEvaluatingJavaScriptFromString:@"navigator.userAgent"];
userAgent = [userAgent stringByAppendingFormat:@" %@" , @"Openjoy SDK/2.00"];
[[NSUserDefaults standardUserDefaults] registerDefaults:@{@"UserAgent":userAgent}];

```

##### 3 代码 \(以下是Objective\_C语言编写， Swift语言编写的在文档末尾\)

```c

-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType{
        *
        * 必选
        */
     if ([request.URL.scheme isEqualToString:@"openjoyscheme"]) {
         NSString *ojHost = request.URL.host;
         NSString *urlString = request.URL.absoluteString;

         NSMutableDictionary *queryItemDic = [@{} mutableCopy];
         NSURLComponents *components = [NSURLComponents componentsWithString:urlString];
         for (NSURLQueryItem *item in components.queryItems) {             
                [queryItemDic setValue:item.value forKey:item.name];
         }

         NSString *jsonString = queryItemDic[@"params"];
         NSDictionary *subDic = [NSJSONSerialization JSONObjectWithData:[jsonString dataUsingEncoding:NSUTF8StringEncoding] options:NSJSONReadingMutableContainers error:nil];
         NSString *callback = subDic[@"callback"];
         if ([ojHost isEqualToString:@"getAppInfo"]) {
             [webView stringByEvaluatingJavaScriptFromString:[callback stringByAppendingString:@"({\"bridgeFuncList\": [\"getAppInfo\", \"runCopyText\" ,\"runShare\" ,\"showShare\" ,\"localRefresh\"]})"]];
         }else if([ojHost isEqualToString:@"runCopyText"]) {
             [[UIPasteboard generalPasteboard] setString:subDic[@"text"]];
             [webView stringByEvaluatingJavaScriptFromString:[callback stringByAppendingString:@"()"]];
         }else if([ojHost isEqualToString:@"showShare"]) {
             /* 页面加载完的分享数据
                分享数据取 subDic
                格式:
                @{@"content":@"分享正文",
                  @"icon":@"分享图片地址",
                  @"logInfo":@"打点信息",
                  @"timelineContent":@"朋友圈正文",
                  @"title":@"分享标题",
                  @"url":@"分享链接"
                 }
             */
         }else if([ojHost isEqualToString:@"runShare"]) {
             /* 页面点击分享
                分享数据取 subDic                
                格式:
                @{@"content":@"分享正文",
                  @"icon":@"分享图片地址",
                  @"logInfo":@"打点信息",
                  @"timelineContent":@"朋友圈正文",
                  @"title":@"分享标题",
                  @"url":@"分享链接"
                 }
             */
         }else if([ojHost isEqualToString:@"localRefresh"]) {
             /* 客户端本地触发刷新积分
                需要自己实现相关代码
             */
         }
         return NO;
     }

    /**
    *  
    调起客户端登录功能：
    1、在商户后台配置登录地址  http://open.kuaiduibao.com/developer/parameter/callback  ->  自定义WEB登录页
    2、通过 url 匹配的方式判断是否需要调用本地登录
    */
    /* 如果需要请去掉注释
    if ([request.URL.absoluteString hasPrefix:@"配置的URL"]) {
        return NO;
    }
    */

    /**
     *  
     调起自己的客户端微信支付：
     */
    /* 如果需要请去掉注释
    if ([request.URL.absoluteString hasPrefix:@"kuaiqiang://bill/pay/call-open"]) {
        NSString *urlString = request.URL.absoluteString;        
        NSMutableDictionary *queryItemDic = [@{} mutableCopy];    
        NSURLComponents *components = [NSURLComponents componentsWithString:urlString];            
        for (NSURLQueryItem *item in components.queryItems) {             
            [queryItemDic setValue:item.value forKey:item.name];
        }
        return NO;
    }
    */
      return YES;
}
```

##### 4 使用第三方支付的额外设置

如果使用了第三方支付工具，例如 微信支付\/支付宝支付，则需要将支付使用的URL Schemes列为白名单,才可正常调用支付App。具体是在“Info.plist”里增加如下代码：

```c
 <key>LSApplicationQueriesSchemes</key>
 <array>

 <!-- 微信 URL Scheme 白名单-->
 <string>weixin</string>

 <!-- 支付宝 URL Scheme 白名单-->
 <string>alipay</string>

 </array>
```

### 示例代码

* 下载源码示例 [Demo下载](http://open.kuaiduibao.com/download/url?type=12)

### 额外说明

* 调起其他客户端功能也是同样的思路，比如赚取积分功能等

### Swift代码

```c

func webView(_ webView: UIWebView, shouldStartLoadWith request: URLRequest, navigationType: UIWebViewNavigationType) -> Bool {
 /** 必选
 */
 if request.url?.scheme == "openjoyscheme" {
     let ojHost = request.url?.host
     let queryItemDic = NSMutableDictionary.init()
     let components = NSURLComponents.init(string: (request.url?.absoluteString)!)
     for item in (components?.queryItems)! {
         queryItemDic.setValue(item.value, forKey: item.name)
     }

     let jsonString : String = queryItemDic["params"] as! String;
     let data : Data = jsonString.data(using: String.Encoding.utf8)!
     var subDic = try! JSONSerialization.jsonObject(with: data, options:JSONSerialization.ReadingOptions.mutableContainers) as! Dictionary<String, Any>
     let callback = subDic["callback"] != nil ? subDic["callback"] as! String : ""

     if ojHost == "getAppInfo" {
         webView.stringByEvaluatingJavaScript(from: (callback ).appending("({\"bridgeFuncList\": [\"getAppInfo\", \"runCopyText\" ,\"runShare\" ,\"showShare\" ,\"localRefresh\"]})"))
     }else if ojHost == "runCopyText" {
         UIPasteboard.general.string = subDic["text"] as! String?
         webView.stringByEvaluatingJavaScript(from: (callback ).appending("()"))
     }else if ojHost == "showShare" {
         /*页面加载完的分享数据
           分享数据取 subDic
           格式:
           @{  @"content":@"分享正文",
               @"icon":@"分享图片地址",
               @"logInfo":@"打点信息",
               @"timelineContent":@"朋友圈正文",
               @"title":@"分享标题",
               @"url":@"分享链接"
            }
         */
     }else if ojHost == "runShare" {
         /*页面点击分享
           分享数据取 subDic
           格式:
           @{  @"content":@"分享正文",
               @"icon":@"分享图片地址",
               @"logInfo":@"打点信息",
               @"timelineContent":@"朋友圈正文",
               @"title":@"分享标题",
               @"url":@"分享链接"
             }
          */
     }else if ojHost == "localRefresh" {
           /* 客户端本地触发刷新积分
              需要自己实现相关代码
           */
     }
    return false
    }

     /**
     *
     调起客户端登录功能：
     1、在商户后台配置登录地址 http://open.kuaiduibao.com/developer/parameter/callback -> 自定义WEB登录页
     2、通过 url 匹配的方式判断是否需要调用本地登录
     */
     /* 如果需要请去掉注释
     if (request.url?.absoluteString.hasPrefix("配置的URL"))! {
         return false
     }
     */

     /**
     *
     调起自己的客户端微信支付：
     */
     /* 如果需要请去掉注释
     if (request.url?.absoluteString.hasPrefix("kuaiqiang://bill/pay/call-open"))! {
         let queryItemDic = NSMutableDictionary.init()
         let components = NSURLComponents.init(string: (request.url?.absoluteString)!)
         for item in (components?.queryItems)! {
             queryItemDic.setValue(item.value, forKey: item.name)
         }
         return false
     }
     */
     return true;
}
```

