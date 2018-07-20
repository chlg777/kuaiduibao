# Android H5接入

* 本文档使用于Android开发人员快速接入

* 需要注意webview的相关配置

### 1. manifest清单文件的配置：

manifest的配置主要包括添加权限，以下权限缺一不可

```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.MEDIA_CONTENT_CONTROL" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```

Android 6.0之后，设备信息部分获取有变动,需要在运行时获取权限申请。  
比如我们有一个读取文件功能，需要写SD卡的权限，我们在写入之前检查是否有WRITE\_EXTERNAL\_STORAGE权限，没有则申请权限

```java
if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
   //申请WRITE_EXTERNAL_STORAGE权限
   ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, WRITE_EXTERNAL_STORAGE_REQUEST_CODE);
 }
```

小技巧：当xml中的targetSdkVersion=x\(x&lt;23\)时候, 可以正常获取信息\(相当于跳过了6.0权限检查\)

### 1.2 配置浏览器的UserAgent

* 必须追加字符串"Openjoy SDK\/2.00"到webview的UA

```java
//示例代码
webSettings.setUserAgentString(webSettings.getUserAgentString() + " OpenJoy SDK/2.00");
```

### 1.3 设置一些必须私有变量图片选择

```java
private boolean blockLoadingNetworkImage;//设置图片阻塞加载，让网页更快显示
private ValueCallback<Uri> mUploadMessageFirst;//上传图片
private ValueCallback<Uri[]> mUploadMessageSecond;//上传图片（android api 5.0后属性）
private JSONObject mJsonObject;//分享的json数据
private static boolean isMoreFive = false;
private static boolean isLessFive = false;
String compressPath = "";
String imagePaths;
Uri cameraUri;
private String htmlPath;//加载的url （传入的url）
private final int OPEN_CARCME = 888;//打开相机
private final int OPEN_PIC = 889;//打开本地文件
private String[] functionList = {"runCopyText", "showShare", "runShare", "getAppInfo", "localRefresh"};//通知webview有多少可以调用的方法集合
```

### 2. webview优化设置

* 开发者可以使用本身项目加载网页的activity，需要注意的是webview需要有如下优化，如已经有可以不需要考虑

```java
/**
     *必选
     **/
    private void setWebView() {
        if (Build.VERSION.SDK_INT >= 19) {//大于19开启硬件加速
           mWebView.setLayerType(View.LAYER_TYPE_HARDWARE, null);
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {//开启软件加速
           mWebView.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
        }
        webSettings.setUseWideViewPort(true);//关键点     webSettings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NARROW_COLUMNS);
        webSettings.setAllowFileAccess(true);// 设置可以访问文件
        webSettings.setBlockNetworkImage(true);//将图片下载阻塞, 通过图片的延迟载入，让网页能更快地显示
        blockLoadingNetworkImage = true;
        webSettings.setRenderPriority(WebSettings.RenderPriority.HIGH);//提高渲染的优先级
        webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);
        webSettings.setDomStorageEnabled(true);
        webSettings.setDatabaseEnabled(true);
        webSettings.setDefaultTextEncodingName("utf-8");//设置编码
        webSettings.setJavaScriptEnabled(true);//设置支持js
        webSettings.setSupportZoom(false);//不能缩放
        webSettings.setBuiltInZoomControls(false);//设置不能缩放
        webSettings.setJavaScriptCanOpenWindowsAutomatically(true);//支持js打开新的窗口
        webSettings.setUserAgentString(webSettings.getUserAgentString() + "Openjoy SDK/2.00");//UA设置 保证桥的调用
        webSettings.setAllowFileAccess(true);
        //WebChromeClient是辅助WebView处理Javascript的对话框，网站图标，网站title，加载进度等 必须有如下设置 调用拍照和图片上传功能，如不设置将不能使用部分功能
        mWebView.setWebChromeClient(new WebChromeClient() {
            // For Android 3.0+
            public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType) {
                if (mUploadMessageFirst != null) {
                    return;
                }
                mUploadMessageFirst = uploadMsg;
                selectImage();
                isLessFive = true;
            }
            // For Android < 3.0
            public void openFileChooser(ValueCallback<Uri> uploadMsg) {
                openFileChooser(uploadMsg, "");
            }

            // For Android > 4.1.1
            public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture) {
                openFileChooser(uploadMsg, acceptType);
            }
            //For Android >5.0
            public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
                if (mUploadMessageSecond != null) return true;
                mUploadMessageSecond = filePathCallback;
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        selectImage();
                        isMoreFive = true;
                    }
                });
                return true;
            }
        });
       //WebViewClient就是帮助WebView处理各种通知、请求事件的，具体来说包括：
        mWebView.setWebViewClient(new WebViewClient() {
            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
                webSettings.setBlockNetworkImage(false);//页面显示出来后加载图片
            }
            /**
             * 只需要重新此方法就会在当前页面打开点击的链接
             * @param view
             * @param url
             * @return
             */
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
             return shouldOverrideUrlByKuaiDuibao(view, url);
            }
        });
        mWebView.addJavascriptInterface(new JsAndAndroid(), "openJoyBridge");//添加网页对js和java代码的调用 代码请见文后的JsAndAndroid 代码
    }
```

### 2.1 上传图片方法（打开文件和照相机）

```java
/**
     * 将map集合装换为字符传
     *
     * @return
     */
    private String mapToString() {
        Map<String, Object> map = new HashMap<>();
        map.put("bridgeFuncList", functionList);
        return OpenJoyTool.map2json(map).toString();
    }

    /**
     * 打开照相机
     */
    private void openCarcme() {
        Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        imagePaths = Environment.getExternalStorageDirectory().getPath()
                + "/fuiou_wmp/temp/"
                + (System.currentTimeMillis() + ".jpg");
        // 必须确保文件夹路径存在，否则拍照后无法完成回调
        File vFile = new File(imagePaths);
        if (!vFile.exists()) {
            File vDirPath = vFile.getParentFile();
            vDirPath.mkdirs();
        } else {
            if (vFile.exists()) vFile.delete();
        }
        cameraUri = Uri.fromFile(vFile);
        intent.putExtra(MediaStore.EXTRA_OUTPUT, cameraUri);
        startActivityForResult(intent, OPEN_CARCME);
    }

    /**
     * 拍照结束后
     */
    private void afterOpenCamera() {
        File f = new File(imagePaths);
        addImageGallery(f);
        Log.e("图片加载路径：", imagePaths);
        //进行图片的压缩
        cameraUri = Uri.fromFile(OpenJoyTool.compressFile(f.getPath(), compressPath));

    }

    /**
     * 解决拍照后在相册中找不到的问题
     */
    private void addImageGallery(File file) {
        ContentValues values = new ContentValues();
        values.put(MediaStore.Images.Media.DATA, file.getAbsolutePath());
        values.put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg");
        getContentResolver().insert(
                MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values);
    }

    /**
     * 本地相册选择图片
     */
    private void chosePic() {
        OpenJoyTool.delFile(compressPath);
        Intent innerIntent = new Intent(Intent.ACTION_PICK); // "android.intent.action.GET_CONTENT"
        String IMAGE_UNSPECIFIED = "image/*";
        innerIntent.setType(IMAGE_UNSPECIFIED); // 查看类型
        Intent wrapperIntent = Intent.createChooser(innerIntent, null);
        startActivityForResult(wrapperIntent, OPEN_PIC);
    }
```

### 2.2 对打开照相机和文件返回结果处理

* 重写onActivityResult\(\)并做如下处理：

```java
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        //5.0以下的
        if (resultCode == Activity.RESULT_OK && (requestCode == OPEN_PIC || requestCode == OPEN_CARCME)) {
            if (isLessFive) {
                isLessFive = false;
                if (null == mUploadMessageFirst)
                    return;
                Uri uri = null;
                if (requestCode == OPEN_CARCME) {
                    afterOpenCamera();
                    uri = cameraUri;
                } else if (requestCode == OPEN_PIC && data != null) {
                    uri = data.getData();
                }
                mUploadMessageFirst.onReceiveValue(uri);
                mUploadMessageFirst = null;
            } else if (isMoreFive) {
                isMoreFive = false;
                if (mUploadMessageSecond == null) return;
                Uri[] dataUris;
                try {
                    if (requestCode == OPEN_CARCME) {
                        afterOpenCamera();
                        dataUris = new Uri[]{cameraUri};
                    } else {
                        dataUris = new Uri[]{Uri.parse(data.getDataString())};
                    }
                    mUploadMessageSecond.onReceiveValue(dataUris);
                    mUploadMessageSecond = null;
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        } else {
            if (mUploadMessageFirst != null) {
                mUploadMessageFirst.onReceiveValue(null);
                mUploadMessageFirst = null;
            } else if (mUploadMessageSecond != null) {
                mUploadMessageSecond.onReceiveValue(null);
                mUploadMessageSecond = null;
            }
        }
    }
```

### 3. 图片上传工具类添加

* 因为拍照上传图片需要用到图片压缩功能，需添加一个工具类

  [OpenJoyTool.java 点击下载](http://file.mmzhuli.com/8da348da17206376bd40b1e04de6c4c4.zip)

### 示例代码

* 下载源码示例 [Demo下载](http://open.kuaiduibao.com/download/url?type=11)

### 额外说明

* webview调用本地java的代码声明

```java
/**
     * js可以调用Android本地方法的集合  必须要添加
     */
    public class JsAndAndroid {

        /**
         * 手动点击分享
         *
         * @param string
         */
        @JavascriptInterface
        public void runShare(final String string) {//分享数据
            mWebView.post(new Runnable() {
                @Override
                public void run() {
                    JSONObject jsonObject = null;
                    try {
                        jsonObject = new JSONObject(string);
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                    if (jsonObject != null)
                        new AlertDialog.Builder(OpenJoyActivity.this)
                                .setTitle("页面分享信息")
                                .setItems(new String[]{"分享标题：{ title: " + jsonObject.optString("title") + " }", "分享正文： { content ：" + jsonObject.optString("content") + " }",
                                        "朋友圈分享正文:{ timelineContent: " + jsonObject.optString("timelineContent") + "}", "分享图片地址：{ icon: " + jsonObject.optString("icon") + " }", "分享链接：{ url: " + jsonObject.optString("url") + " }"}, null)
                                .setNegativeButton("确定", null)
                                .show();
                }
            });
        }

        /**
         * 自动显示分享按钮
         *
         * @param string
         */
        @JavascriptInterface
        public void showShare(final String string) {
            try {
                mJsonObject = new JSONObject(string);
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
        /**
        *复制券码
        */
        @JavascriptInterface
        public void runCopyText(final String code) {
            mWebView.post(new Runnable() {
                @Override
                public void run() {
                    try {
                        JSONObject jsonObject = new JSONObject(code);
                        ClipboardManager clip = (ClipboardManager) getSystemService(Context.CLIPBOARD_SERVICE);
                        clip.setText(jsonObject.optString("text")); // 复制
                        StringBuilder callbackString = new StringBuilder();
                        callbackString.append("javascript:" + jsonObject.optString("callback") + "()");
                        mWebView.loadUrl(callbackString.toString());
                    } catch (JSONException e) {
                        e.printStackTrace();
                    }
                }
            });

        }

        /**
         * 获取App信息
         *
         * @param string
         */
        @JavascriptInterface
        public void getAppInfo(final String string) {
            mWebView.post(new Runnable() {
                @Override
                public void run() {
                    try {
                        JSONObject jsonObject = new JSONObject(string);
                        if (jsonObject != null) {
                            StringBuilder callbackFunc = new StringBuilder();
                            callbackFunc.append("javascript:").append(jsonObject.optString("callback")).append("('").append(mapToString()).append("')");
                            if (mWebView != null) {
                                mWebView.loadUrl(callbackFunc.toString());
                            }
                        }
                    } catch (JSONException e) {
                        e.printStackTrace();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            });
        }
         /**
         * 客户端本地触发刷新积分
         *
         */
        @JavascriptInterface
        public void localRefresh(final String string) {
           //开发者调用刷新积分接口
        }
    }
```

* webview拦截url代码\(重要\)

```java
    /**
     * 拦截url请求，根据url结尾执行相应的动作。 （重要）
     *
     * @param view
     * @param url
     * @return
     */
    protected boolean shouldOverrideUrlByKuaiDuibao(WebView view, String url) {
        Uri uri = Uri.parse(url);
        if (this.htmlPath.equals(url)) {
            view.loadUrl(url);
            return true;
        }
        // 处理电话链接，启动本地通话应用。
        if (url.startsWith("tel:")) {
            Intent intent = new Intent(Intent.ACTION_DIAL, uri);
            startActivity(intent);
            return true;
        }
        /**
         * 调起客户端登录功能：
         * 1、在商户后台配置登录地址 https://open.kuaiduibao.com/developer/parameter/callback -> 自定义WEB登录页
         * 2、通过 url 匹配的方式判断是否需要调用本地登录
         * 如果需要请去掉以下注释
         **/

        if (uri.getPath().contains("配置的URL")) { //调用本地登录
            return true;
        } else if (url.endsWith(".apk") || url.contains(".apk?")) { // 支持应用链接下载
            Intent viewIntent = new Intent(Intent.ACTION_VIEW, uri);
            startActivity(viewIntent);
            return true;

        }
        //判断非http连接时尝试发出系统级intent（比如tbopen等打开第三方应用）
        if (!url.startsWith("http://") && !url.startsWith("https://")) {
            final PackageManager packageManager = this.getPackageManager();
            final Intent intent = new Intent(Intent.ACTION_VIEW);
            intent.setData(uri);
            List<ResolveInfo> list = packageManager.queryIntentActivities(intent, PackageManager.MATCH_DEFAULT_ONLY);
            if (list.size() > 0) {//可以处理此intent的情况
                startActivity(intent);
                return true;
            }

            return false;
        }

        /**
         *调起自己的客户端微信支付：
         * 如果需要请去掉注释
         */

        /**
         if (Uri.parse(url).getPath().contains("kuaiduibao://bill/pay/call-open")) {
         String type = url.substring(url.indexOf("?") + 1, url.length());
         String weiXinParams[] = type.split("&");
         Map<String, String> WXMap = new HashMap<>();
         for (int i = 0; i < weiXinParams.length; i++) {
         String params[] = weiXinParams[i].split("=");
         WXMap.put(params[0], params[1]);
         }
         Log.e("OpenJoyActivity", WXMap.toString());
         // 调起自己的客户端微信支付：
         // 注释以下代码调用客户端微信支付

         IWXAPI api = WXAPIFactory.createWXAPI(this, "在后台申请的appid", false);
         api.registerApp(APP_ID);
         String partnerid = WXMap.get("partnerid")//
         String nonceStr = WXMap.get("nonceStr");
         String packageValue = WXMap.get("package");
         String prepay_id = WXMap.get("prepay_id");
         String timeStamp = WXMap.get("timeStamp");
         String paySign = WXMap.get("paySign");
         String appId=WXMap.get("appId");
         PayReq req = new PayReq();
         req.appId = appId;
         req.partnerId = partnerid;
         req.prepayId = prepay_id;
         req.nonceStr = nonceStr;
         req.timeStamp = timeStamp;
         req.packageValue = packageValue;
         req.sign = paySign;
         req.extData = "app data";
         api.sendReq(req);

         return true;
         }
         */

        view.loadUrl(url);
        return false;
    }
```

### 其他

* 调起客户端登录功能

* 配置的登录链接一般为H5页面，在客户端内通过拦截 url 的方式，调起客户端登录组件；非客户端即可跳转到 H5 页面进行登录

* 在商户后台配置登录页面链接 [配置登录](https://open.kuaiduibao.com/developer/parameter/callback)

* 实现 webview中的 shouldOverrideUrlLoading 方法，判断是否为自定义的登录页面

* 调起客户端登录页面

* 调起其他客户端功能也是同样的思路，比如赚取积分功能等

* 调起微信支付功能



