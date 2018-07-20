# Android客户端基础
* 本文档使用于Android开发人员快速接入
### 1.场景化接入关键代码
##### 1.初始化：
在需要调起的场景化的Activity onCreate()方法里面初始化以下代码：


```java
sceneBasePopwindow = SceneBasePopwindow.getInstance();
sceneBasePopwindow.setIsDebug(true);//初始化设置
sceneBasePopwindow.init(this, "APPID");//这里填写后台配置的APPID
 ```
 ##### 2.注册场景示例代码：


```java
  RequestBean paramsBean = new RequestBean();
  paramsBean.setApp_id("1001237");
  paramsBean.setApp_uid("114922");
  paramsBean.setEvent_code("registered");
  paramsBean.setGender(1);
  paramsBean.setExtra_param("paramsBean");
  paramsBean.setImei(myimei);
  sceneBasePopwindow.trigger(paramsBean);
```
##### 3.登录场景示例代码：


```java
  paramsBean = new RequestBean();
  paramsBean.setApp_id("1001237");
  paramsBean.setApp_uid("114922");
  paramsBean.setEvent_code("login");
  paramsBean.setGender(1);
  paramsBean.setExtra_param("paramsBean");
  paramsBean.setImei(myimei);
  sceneBasePopwindow.trigger(paramsBean);
```
### 2.需要在项目工程里面添加几个java文件
##### 请求文件包括RequestBean.java, SceneBasePopwindow.java,SceneToolUtil.java 3个文件
[scene 点击下载](http://file.mmzhuli.com/701ded4ff029205f939da49f8090d7cd.zip)
