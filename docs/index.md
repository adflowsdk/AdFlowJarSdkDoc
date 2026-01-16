# AdFlowSkd接入文档

当前版本：V1.0.01
请按以下说明接入。

## 一、项目配置
### 1、添加jar到项目

1.将jar包复制到app下的libs目录下。注意如果有旧版本的话请先删除。

### 2、添加依赖

1.在app包下的build.gradle的dependencies里添加以下代码：
```java
    dependencies {
        implementation fileTree(dir: "libs", include: ["*.jar", "*.aar"])
    }
```

### 3、权限说明

1.在app包的AndroidManifest.xml添加以下基本网络权限
```java
    <!--添加网络权限-->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

2.设置允许使用http
    在AndroidManifest.xml的application节点内添加以下代码:
```java
    android:usesCleartextTraffic="true"
```

### 4、混淆

1.添加以下规则到项目的混淆文件proguard-rules.pro内：
```java
-keeppackagenames com.adflow.adflowads.**
-keep public class com.adflow.adflowads.client.** {
    public *;
}
-keepattributes *Annotation*
-keepattributes Signature
-keepattributes Exceptions

# 2. 保留自定义注解类
-keep class com.adflow.adflowads.common.AdFlowNonNull
-keep class com.adflow.adflowads.common.AdFlowNullable
```

## 二、代码配置

### 1、sdk初始化
1.如果app内没有application类的话，则需要新建一个类继承自Application. 假设类名为 MainApplication, 即：
```java
public class MainApplication extends Application {}
```

2.在MainApplication的onCreate接口内添加以下初始化代码
```java
AdFlowInitialize.initialize(this, app_id,
    channel_id, new AdFlowInitializeListener() {
        @Override
        public void onInitialzeSuccess() {
        }

        @Override
        public void onInitialzeFail(String message) {
        }
    });
```
	app_id和channel_id需要联系对接人员给出。

3. MainApplication文件头部需要引入接口文件：
```java
import com.adflow.adflowads.client.AdFlowInitialize;
import com.adflow.adflowads.client.AdFlowInitializeListener;
           
```
4. 如果处于测试阶段，建议打开日志开关，将使用第三方链接测试，不影响正常任务链接。
切记：发布时请删除或者设置为false。否则发布后将没有数据。
```java
AdFlowInitialize.openDebugLog(true);//发布时必须设置false或者删除该行代码
```
Note: 发布前请删除该接口
        
### 2. 初始化广告
1.在接入的类的头部添加以下引用：
```java
import com.adflow.adflowads.client.AdFlowAdError;
import com.adflow.adflowads.client.AdFlowInterstitialAdListener;
import com.adflow.adflowads.client.AdFlowInterstitial;
```
2.新建一个类成员变量：
```java
AdFlowInterstitial mAdFlowInterstitialAd;
```
3.初始化成员变量，在load广告之前调用接口initInterstitial完成初始化
```java
private void initInterstitial(){
    mAdFlowInterstitialAd = new AdFlowInterstitial(this);//this有activity优先传activity，没有则传ApplicationContext
    mAdFlowInterstitialAd.setAdListener(new AdFlowInterstitialAdListener() {
        @Override
        public void onInterstitialAdLoaded() {
        }

        @Override
        public void onInterstitialAdLoadFail(AdFlowAdError var1) {
        }

        @Override
        public void onInterstitialAdClicked() {
        }

        @Override
        public void onInterstitialAdShowedSuccess() {
        }

        @Override
        public void onInterstitialAdShowFail(AdFlowAdError var1) {
        }

        @Override
        public void onInterstitialAdClose() {
        }
		@Override
		public void onInterstitialAdRealShowedSuccess() {
		}

		@Override
		public void onInterstitialAdRealShowFail(AdFlowAdError var1) {
		}
    });
}
```
此回调接口可判断广告的多个状态，加载成功/失败，展示成功/失败，关闭广告
### 3. 预加载广告
1.添加以下代码进行广告预加载
```java
private void loadAd() {
    if (mAdFlowInterstitialAd == null) {
        return;
    }
    //如果要添加子渠道信息需要添加以下4行代码,否则可不加
    Map<String, Object> localMap = new HashMap<>();
    localMap.put("channel", "1008");;
    localMap.put("sub_channel", "3006");
    mAdFlowInterstitialAd.setLocalExtra(localMap);
    //这行代码必须保留
    mAdFlowInterstitialAd.loadAd();
}
```
2.如果预加载广告成功，则会进入上一步的回调接口onInterstitialAdLoaded内
    
 
### 4. 显示广告
1.调用接口show展示广告
```java
private void showAd() {
if (mAdFlowInterstitialAd != null && mAdFlowInterstitialAd.isAdReady()) {
        mAdFlowInterstitialAd.showAd(this);//this必须为activity
    }
}
```

### 5. 释放资源
1.在退出场景或者关闭activity的时候，需在onDestroy内调用下面的接口释放资源
```java
private void destroyInterstitial() {
    if (mAdFlowInterstitialAd != null) {
        mAdFlowInterstitialAd.destory();
    }
    mAdFlowInterstitialAd = null;
}
```

## 三、错误码说明

        | 错误码    | 说明               
        | 4001      | 请检查手机网络       
        | 4002      | 不允许调试，直接run   
        | 4003      | 任务获取后超时未显示 
        | 4004      | 传入正显示的activity 
        | 4005      | 加载广告失败        
        | 5001      | 没有任务            
        | 6001      | 没有广告           
        | 其他      | 预加载失败        
 
