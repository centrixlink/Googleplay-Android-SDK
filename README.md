# Centrixlink Android SDK Googleplay版本 集成文档

### 版本支持及依赖

* Centrixlink 支持 Android 4.0以上系统版本（API LEVEL 14+）
* 第三方库依赖 Google Play Services (必选), libammsdk.jar	(可选)
* 申请：[App ID、App Key](https://dashboard.centrixlink.com/login)

### 集成说明

### 1 将下载的Centrixlxink SDK库文件拷贝到项目应用的libs目录下

```
  libs
	|-----Centrixlink_[version].jar
	|-----libammsdk.jar				(集成微信分享功能，可选)
```
### 2 在build.gradle中引入 Google Play Services的依赖
```
    compile 'com.google.android.gms:play-services-ads:9.8.0'
```

### 3 更新 AndroidManifest.xml
```	xml
<manifest>

	...
	<!-- 权限 -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />


	<application>

		...

    <!-- 广告 Activity -->
        <activity android:name="com.centrixlink.SDK.FullScreenADActivity"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize|screenLayout|smallestScreenSize"
            android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
            android:process=":adprocess"
            tools:ignore="InnerclassSeparator" />

        <activity android:name="com.centrixlink.SDK.ResizedVideoADActivity"
            android:configChanges="keyboard|keyboardHidden|orientation|screenSize|screenLayout|smallestScreenSize"
            android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen"
            android:hardwareAccelerated="true"
            android:process=":adprocess"
            tools:ignore="InnerclassSeparator" />
      <!-- 广告 service -->
        <service
            android:name="com.centrixlink.SDK.service.CentrixlinkService"
            android:exported="false" />


    </application>

</manifest>

```




### 4 集成说明


#### 4.1 SDK初始化
``` Java
private CentrixlinkVideoADListener eventListener;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    ....
    //实际开发中请务必使用自己申请的 AppID 和 AppKey 
    final String appID = "FQ11tkfWJ4";
    final String appKey = "MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC6WzCw9MWQjwUH76KR+cjr/CdCyQ1UqQRqsMFSRmhM2XHPpXUp4v+vAL984P8xhZ/QLOMIULcLfuegqrzEm0lobJy/dLMy+e18ucR/z1lr6gXnItTwqliJfmNFQOOpYGs8OprucdYqtBl7M4keVDBPYOpVkBTSGr6HxKquZyA9tQIDAQAB";

    //init SDK
    final Centrixlink centrixlink =   Centrixlink.sharedInstance();

    //SDK启动
    centrixlink.startWithAppID(this,appID,appKey);

}

```


#### 4.2 视频广告


##### 4.2.1 视频广告播放回调事件设置

``` Java
final Centrixlink centrixlink =   Centrixlink.sharedInstance();
CentrixlinkVideoADListener eventListener =  new CentrixlinkVideoADListener() {

            @Override
            public void centrixLinkVideoADWillShow(Map map){
                //视频广告即将播放
                //key:"ADID",value:String
            }

            @Override
            public void centrixLinkVideoADDidShow(Map map) {
                //视频广告播放开始
                //key:"ADID",value:String
            }

            @Override
            public void centrixLinkHasPreloadAD(boolean isPreloadFinished) {
                //本地是否有预加载的广告 false表示没有，true表示有
            }

            @Override
            public void centrixLinkVideoADShowFail(Map map){
            //视频广告播放失败
            //key:"error", value:AD_PlayError
            /* AD_PlayError
                100 广告的播放间隔时间不满足条件
                101 本地没有可播放广告
                105 当前正在播放其它广告
                106 处于静默状态
                107 本地广告资源不可用
                108 当前用户播放超限
            */
            }

            @Override
            public void centrixLinkVideoADAction(Map map) {
                //视频广告触发了点击事件
                //key:"ADID", value:String
            }

            @Override
            public void centrixLinkVideoADClose(Map map) {
                //视频广告关闭
                //key:"ADID", value:String
                //key:"playFinished", value:boolean;true表示播放完成
                //key:"isClick", value:boolean;true表示触发

            }

        };
        centrixlink.addEventListeners(eventListener);

```
##### 4.2.2 全屏播放视频广告

``` Java
final Centrixlink centrixlink =   Centrixlink.sharedInstance();

/**
 @param Activity activity 当前activity
 */

centrixlink.playAD(activity);

/**如果你需要配置Server to Server 的透传参数，可以使用AdConfig 进行透传配置**/
AdConfig config = new AdConfig();
config.setOptionKeyExtra1("****");//你可以使用保留的八个自定义参数 optionKeyExtra1-optionKeyExtra8来配置你的自定义参数
Bundle bundle = new Bundle();//或者你可以使用bundle 通过 key value的形式传入你的自定义参数,我们支持所有类型参数和list类型的参数
 bundle.putString("Your parameter name","Your parameter value");
bundle.putInt("Your parameter name1",1);
ArrayList<String> list = new ArrayList<>();
list.add("1");
bundle.putStringArrayList("Your parameter name2",list);
config.setBundle(bundle);
centrixlink.playAD(mActivity,config);

```

##### 4.2.3 检查当前是否可以播放视频广告

```Java
final Centrixlink centrixlink =   Centrixlink.sharedInstance();
/*
 * 检查当前是否可以播放视频广告
 * 功能等同于预加载状态centrixLinkHasPreloadAD回调方法
*/
centrixlink.hasPreload();
```


##### 4.2.4 设置视频广告显示方向是否跟随应用方向

```Java
/**
 设置是否跟随应用方向

 @param boolead enable 默认值为true;
 */
 centrixlink.setEnableFollowAppOrientation(enable);

```
#### 4.3 开屏图片广告及监听设置

``` Java
final Centrixlink centrixlink =   Centrixlink.sharedInstance();

centrixlink.playSplashAD(this, new SplashADEventListener() {

            @Override
            public void centrixlinkSplashADDidShow(Map map) {
            //开屏图片广告已经显示
            //key:"ADID", value:String

            }

            @Override
            public void centrixlinkSplashADAction(Map map) {
            //开屏图片广告点击事件
            //key:"ADID", value:String


            }

            @Override
            public void centrixlinkSplashADShowFail(AD_PlayError error) {
            //开屏图片广告显示失败
            /* AD_PlayError
            201    本地无缓存
	         */
            }

            @Override
            public void centrixlinkSplashADClosed(Map map) {
            //开屏图片广告关闭
            //key:"ADID", value:String
            //key:"isClick", value: boolean
            //key:"isSkip", value: boolean
            //key:"playFinished", value: boolean


            }


            @Override
            public void centrixlinkSplashADSkip(Map map) {
            //开屏图片广告跳过
            //key:"ADID", value:String

            }

        });

```

#### 4.4 Activty生命周期与SDK关联处理

``` Java
@Override
protected void onDestroy() {
    super.onDestroy();
    final Centrixlink centrixlink =   Centrixlink.sharedInstance();

    //重置SDK，防止内存泄漏
    centrixlink.removeEventListeners(eventListener);
    centrixlink.setDebugCallBack(null);
}
```

#### 4.5 其它接口

``` Java
 //微信朋友圈分享（分享endcard页面，此项为可选）
 IWXAPI api = WXAPIFactory.createWXAPI(this, "WX APP ID");

 api.registerApp("WX APP ID");
 centrixlink.setWXAPIObject(api);

```

#### 4.6 混淆设置
```
# 对Centrixlink SDK的混淆处理
-dontwarn com.centrixlink.**
-keep public class com.centrixlink.**  { *; }


# 对微信SDK的混淆处理
-keep class com.tencent.mm.sdk.** {
   *;
}

```
