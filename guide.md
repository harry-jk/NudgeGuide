## Contents
- [Basic Integration](#basic-integration)
  - [Installation](#installation)
  - [Start Session](#start-session)
  - [In-App Messaging](#in-app-messaging)
  - [Push Messaging](#push-messaging)
  - [Test Device Registration](#test-device-registration)
- [IAP, Reward and Sales Promotion](#iap-reward-and-sales-promotion)
  - [In-App Purchase Tracking](#in-app-purchase-tracking)
  - [Give Reward](#give-reward)
  - [Sales Promotion](#sales-promotion)
- [Dynamic Targeting](#dynamic-targeting)
  - [Custom Parameter](#custom-parameter)
  - [Marketing Moment](#marketing-moment)
- [Advanced](#advanced)
  - [Timeout Interval](#timeout-interval) 
- [Reference](#reference)
  - [Deep Link](#deep-link)
  - [Cross Promotion Configuration](#cross-promotion-configuration)
  - [Image Push Notification](#image-push-notification)
  - [Proguard Configuration](#proguard-configuration)
- [Troubleshooting](#troubleshooting)
- [Release Notes](#release-notes)

* * *

## Basic Integration

### Installation

아래 링크를 통해 _COCOS2DX Plugin_을 다운로드 합니다.

[cocos2dx Plugin 다운로드]()


환경변수에 __COCOS2DX_ROOT__ 가 등록되어 있는지 확인합니다.

copy_plugin.sh를 실행합니다.

아래의 구성요소들이 COCOS2D-X SDK폴더안에 있는 plugin폴더에 추가되어야 합니다.

plugins/  

    nudge  
protocols/include/  

	NudgeAgent.h  
    NudgeException.h  
    NudgePurchase.h  
    NudgeRewardItem.h  
protocols/platform/android/  

	NudgeAgent.cpp  
    NudgeException.cpp  
    NudgePurchase.cpp  
    NudgeRewardItem.cpp
protocols/platform/ios/  

	NudgeAgent.mm  
    NudgeException.mm  
    NudgePurchase.mm  
    NudgeRewardItem.mm


이제 각 플랫폼에 맞게 설치 작업을 진행합니다.


#### iOS

COCOS2D-X SDK 폴더에서 아래의 구성요소를 프로젝트에 Drag & Drop 하여 추가합니다.  
(PluginProtocol.xcodeproj의 경우 이미 등록된 파일이 있다면 생략합니다.)


plugin/protocols/proj.ios/
	
    PluginProtocol.xcodeproj  
plugin/plugins/nudge/proj.ios/  

    PluginNudge.xcodeproj

<img src="import_projects.png" width="240" />

PluginProtocol.xcodeproj에 아래와 같이 파일을 추가합니다.
    - include폴더에 플러그인 해더 파일을 추가합니다.
    		헤더파일 경로 : {COCOS2DX SDK ROOT}/plugin/protocols/include/
            	NudgeAgent.h  
				NudgeException.h  
        		NudgePurchase.h  
        		NudgeRewardItem.h
<img src="import_header.png" width="240" />
	- ios 폴더에 플러그인 소스 파일을 추가합니다.
    		소스파일 경로 : {COCOS2DX SDK ROOT}/plugin/protocols/platform/ios/
            	NudgeAgent.mm  
				NudgeException.mm  
        		NudgePurchase.mm  
        		NudgeRewardItem.mm
<img src="import_source.png" width="240" />


XCode 프로젝트의 Build 설정을를 다음과 같이 수정 합니다.
- Build Settings
	- Other Linker Flags : -ObjC 추가
    - User Header Search Paths : /{COCOS2DX SDK ROOT}/plugin/protocols/include 추가  
<img src="other_linker_flags.png" width="540" />  
<img src="user_header_path.png" width="540" />
- Build Phases
	- Link Binary Libraries
    	- libPluginNudge.a 추가
        - libPluginProtocol.a 추가
        - GameController.framework 추가
        - MediaPlayer.framework 추가
        
<img src="link_lib.png" width="640" />

이후 iOS SDK 설치 가이드의 ['Installation'](https://github.com/adfresca/sdk-ios/blob/master/README.kor.md#installation) 항목을 따라서 설치 작업을 완료합니다.

#### Android

다음 경로에 있는 파일을 다음 설명과 같이 수정합니다.
- {COCOS2D-X SDK ROOT}/plugin/protocols/proj.android/jni/Anroid.mk
```mk
...
LOCAL_SRC_FILES :=\
$(addprefix ../../platform/android/, \
...
NudgeAgent.cpp \
NudgePurchase.cpp \
NudgeRewardItem.cpp \
NudgeException.cpp \
) \
...
```
<img src="plugin_mk.png" width="640" />  

plugin-x빌드를 위해 Android SDK (android-7)을 설치해 주세요.  
<img src="android_sdk_7.png" width="640" />  

install_android.sh를 실행후 지시에 따라 진행합니다.  
(지시에 나오는 모든 경로는 절대경로를 넣어주세요.)  
<img src="install_android.png" width="640" />
<img src="insert_path.png" width="540" />
<img src="select_plugin.png" width="540" />

다음 파일을 수정합니다.
- proj.android/jni/Android.mk
```mk
include $(CLEAR_VARS)
...
$(call import-add-path,$(COCOS2DX_ROOT)/plugin/publish)
...
LOCAL_MODULE := cocos2dcpp_shared
```
<img src="android_mk.png" width="440" />
- proj.android/jni/hellocpp/main.cpp
```cpp
...
#include "PluginJniHelper.h"
#include "NudgeAgent.h"
...
void cocos_android_app_init(JNIEnv* env, jobject thiz) {
    ...

    JavaVM* vm;
    env->GetJavaVM(&vm);
    PluginJniHelper::setJavaVM(vm);
}
```
<img src="main_cpp.png" width="540" />
- proj.android/src/org/cocos2dx/cpp/AppActivity.java
```java
...
import org.cocos2dx.lib.Cocos2dxGLSurfaceView;
import org.cocos2dx.plugin.PluginWrapper;
...
public class AppActivity extends Cocos2dxActivity {
    ...
    @Override
    public Cocos2dxGLSurfaceView onCreateView() {
        Cocos2dxGLSurfaceView glSurfaceView = new Cocos2dxGLSurfaceView(this);
        glSurfaceView.setEGLConfigChooser(5, 6, 5, 0, 16, 8);

        PluginWrapper.init(this);
        PluginWrapper.setGLSurfaceView(glSurfaceView);

        return glSurfaceView;
    }
    ...
}
```
<img src="activity_java.png" width="540" />

### Start Session

이제 플러그인을 적용을 시작하기 위해 몇 가지 간단한 코드를 적용합니다. 첫 번째로 API Key를 설정하고 앱의 실행을 기록하는 StartSession() 메소드를 적용합니다. API Key는 [Dashboard](https://admin.adfresca.com) 사이트에서 등록한 앱을 선택한 후 Overview 메뉴의 Settings - API Keys 버튼을 클릭하여 확인이 가능합니다. 

게임이 실행되는 시점에서 StartSession(const char* apiKey) 메소드를 실행합니다.
-에 추가하는것을 권장합니다.

```cpp
#include "NudgeAgent.h"

void AppDelegate::applicationDidFinishLaunching()
{
...
#if (CC_TARGET_PLATFORM == CC_PLATFORM_IOS)
plugin::NudgeAgent::getInstance()->startSession("YOUR_IOS_API_KEY");
#elif (CC_TARGET_PLATFORM == CC_PLATFORM_ANDROID)
plugin::NudgeAgent::getInstance()->startSession("YOUR_ANDROID_API_KEY");
#endif
...
}

```


### In-App Messaging

인-앱 메시징 기능을 이용하여, 사용자에게 원하는 메시지를 실시간으로 전달할 수 있습니다. 메시지를 전달하고자 하는 시점에 제공되는 load(), show() 메소드만을 호출하여 적용이 가능합니다. 메시지는 전면 interstitial 이미지, 텍스트, 혹은 iframe 웹페이지 형태로 게임 화면에 표시될 수 있습니다. 메시지는 현재 게임을 플레이 중인 사용자가 인-앱 메시징 캠페인의 조건과 매칭된 경우에만 화면에 표시됩니다. 조건에 만족하는 캠페인이 없다면 사용자는 아무런 화면을 보지 않고 자연스럽게 게임 플레이를 이어갑니다. 매칭과 관련한 인-앱 메시징의 다이나믹 타겟팅 기능은 아래의 [Dynamic Targeting](#dynamic-targeting) 항목에서 보다 자세히 설명하고 있습니다.

```cpp
USING_NS_CC;
void AScene::A()
{
	plugin::NudgeAgent* nudge = plugin::NudgeAgent::getInstance();
    nudge->load();
    nudge->show();
}
```

첫 번째로 인-앱 메시징 코드를 적용한 경우, 아래와 같이 테스트 이미지 메시지가 표시됩니다. 해당 이미지를 터치하면 앱스토어 페이지로 이동합니다. 현재 보고 있는 테스트 메시지는 이후 테스트 모드 설정을 변경하여 더이상 보이지 않도록 설정하게 됩니다.

<img src="https://adfresca.zendesk.com/attachments/token/ans53bfy6mwq2e9/?name=4444.png" width="240" />
&nbsp;
<img src="https://adfresca.zendesk.com/attachments/token/ec7byt0qtj00qpb/?name=5555.png" height="240" />

### Push Messaging

푸시 메시징 기능을 이용하여 사용자가 게임을 플레이하지 않을 때에도 언제든 메시지를 전달할 수 있습니다. 아래의 플랫폼 별 적용 과정을 통하여 푸시 메시징 기능을 적용합니다.

#### Android

SDK를 적용하기 이전에 [Google API Console](https://cloud.google.com/console) 사이트에서 프로젝트를 생성하고, [Dashboard](https://admin.adfresca.com) 사이트에 설정할 GCM API Key 및 SDK 적용에 필요한 GCM_SENDER_ID (Project Number) 값을 얻어야 합니다.

'[Android Push Notification 설정 및 적용하기 (GCM)](https://adfresca.zendesk.com/entries/28526764)' 가이드를 참고하여 필요한 값들을 얻습니다.

이제 SDK 적용을 시작합니다.


1) AndroidManifest.xml 내용 추가하기

```xml
<manifest>   
  <application>
      .........
      <receiver android:name="YOUR.PACKAGE.NAME.GCMReceiver"
        android:permission="com.google.android.c2dm.permission.SEND">  
        <intent-filter>
          <action android:name="com.google.android.c2dm.intent.RECEIVE" />
          <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
          <category android:name="YOUR.PACKAGE.NAME" />
         </intent-filter>
      </receiver>
      <service android:name="YOUR.PACKAGE.NAME.GCMIntentService" />  

      <activity android:name="com.adfresca.ads.AdFrescaPushActivity" />
      ..........
   </application>
    ..........
    <permission android:name="YOUR.PACKAGE.NAME.permission.C2D_MESSAGE" android:protectionLevel="signature" />
    <uses-permission android:name="YOUR.PACKAGE.NAME.permission.C2D_MESSAGE" />
    <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
    <uses-permission android:name="android.permission.GET_ACCOUNTS" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
    <uses-permission android:name="android.permission.READ_PHONE_STATE" /> 
    <uses-permission android:name="android.permission.VIBRATE" />
    ..........
</manifest>
```

- GCMReceiver 클래스와 GCMIntentService 클래스는 이미 적용 중인 내용이 있다면 그대로 사용하여 SDK 코드만 추가합니다. 
- 만약 기존에 사용 중인 GCM 클래스가 없다면, 샘플로 제공되는 [GCMReceiver](https://gist.github.com/sunku/29906033dcee764ef022) 및 [GCMIntentService](https://gist.github.com/sunku/05c5e4feb3d0fb8d4088) 소스코드를 참고하여 클래스를 작성합니다.해야 합니다.

2) GCMIntentService 클래스 구현하기

```java
protected void onRegistered(Context context, String registrationId) {
	AdFresca.handlePushRegistration(registrationId);
}

protected void onUnregistered(Context context, String registrationId) {
	AdFresca.handlePushRegistration(null);
}

protected void onMessage(Context context, Intent intent) {
      // Check if this notification is from Nudge
      if (AdFresca.isFrescaNotification(intent)) {

        Class<?> targetActivityClass = AppActivity.class;
        String appName = context.getString(R.string.app_name);
        int icon = R.drawable.icon;
        long when = System.currentTimeMillis();

        // Show this message
        AFPushNotification notification = AdFresca.generateAFPushNotification(context, intent, targetActivityClass, appName, icon, when);
        notification.setDefaults(Notification.DEFAULT_ALL); 
        AdFresca.showNotification(notification);
    }
}
```
- 푸시 메시지에 Deep Link가 포함되어 있다면, 사용자가 메시지 터치 시에 SDK는 해당 URL을 실행합니다.
- 푸시 메시지에 Deep Link가 없다면, SDK는 targetActivityClass를 실행하여 앱을 실행합니다. 
- notification.setSound(uri) 메소드를 이용하면 메시지 수신 시에 사운드 파일을 재생할 수도 있습니다.

3) GCMReceiver 클래스 구현하기

```java
@Override
protected String getGCMIntentServiceClassName(Context context) { 
	return "YOUR.PACKAGE.NAME.GCMIntentService"; 
} 
```

5) Cocos2D-X 환경에서 GCM 적용하기

이제 Cocos2D-X코드에서 GCM Sender ID 값을 플러그인에 설정하여 적용을 완료합니다. 만약 기존에 사용 중인 GCM 서비스가 있어 이미 디바이스의 GCM 푸시 고유 아이디를 알고 있는 경우, setPushRegistrationIdentifier(const char*) 메소드를 호출하여 값만 설정하는 것도 가능합니다.

```cs
void AppDelegate::applicationDidFinishLaunching()
{
	plugin::NudgeAgent* nudge = plugin::NudgeAgent::getInstance();

	nudge->setGCMSenderID(GCM_SENDER_ID);  // Google API Proejct Number (ex: 12345678)
	// nudge->setPushRegistrationIdentifier("DEVICE_GCM_PUSH_REG_ID"); // if you already have it

	nudge->startSession(API_KEY);
}
```

#### iOS

1) APNS 인증서 파일(.p12)을 Dashboard에 등록하기
  - Keychain 툴을 이용하여 .cer 인증서 파일을 .p12로 변환하고 [Dashboard](https://admin.adfresca.com) 사이트에 등록합니다.
  - 보다 자세한 설명은 [iOS Push Notification 인증서 설정 및 적용하기](https://adfresca.zendesk.com/entries/21714780) 가이드를 통하여 확인이 가능합니다.

2) Info.plast 확인하기 / Provision 확인하기
- Nudge는 APNS의 Production 환경만을 지원합니다. 때문에 게임 빌드가 production으로 빌드되어야 정상적인 서비스 이용이 가능합니다.
- Info.plst 파일의 'aps-environment' 값을 'production' 으로 설정되어 있어야 합니다.
- App Store / Ad Hoc release에 사용하는 Provision 인증서를 사용하여 빌드되어야 합니다.

3) AppController.mm 코드 적용하기 

```mm
#import <AdFresca/AdFrescaView.h>

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  ....
  if ([application respondsToSelector:@selector(registerUserNotificationSettings:)]) {
    UIUserNotificationType types = (UIUserNotificationTypeAlert | UIUserNotificationTypeBadge | UIUserNotificationTypeSound);
    UIUserNotificationSettings *notificationSettings = [UIUserNotificationSettings settingsForTypes:types categories:nil];
    [application registerUserNotificationSettings:notificationSettings];
    [application registerForRemoteNotifications];
  } else {
    [application registerForRemoteNotificationTypes:(UIRemoteNotificationTypeBadge | UIRemoteNotificationTypeAlert | UIRemoteNotificationTypeSound)];
  }

  NSDictionary* userInfo = [launchOptions objectForKey:UIApplicationLaunchOptionsRemoteNotificationKey];
  if (userInfo != nil) {
    [self application:application didReceiveRemoteNotification:userInfo];
  }
} 

- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
  [AdFrescaView registerDeviceToken:deviceToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
  if ([AdFrescaView isFrescaNotification:userInfo] && [application applicationState] != UIApplicationStateActive) {
    [AdFrescaView handlePushNotification:userInfo];
  }
} 
```

이로써 푸시 메시징 기능을 위한 적용 작업이 모두 완료되었습니다.

### Test Device Registration

Nudge는 테스트 모드 기능을 지원하여 테스트를 원하는 디바이스에만 원하는 메시지를 전달할 수 있습니다. 이로 인해 SDK가 적용된 앱이 이미 앱스토어에 출시된 경우, 게임 운영팀 혹은 개발팀에게만 새로운 메시지를 전달하여 테스트할 수 있도록 지원합니다.

테스트 기기 등록을 위한 아이디 값은 SDK를 통해 추출이 가능하며 2가지 방법을 지원 합니다.

1. testDeviceId 값을 직접 얻어와서 로그로 출력하는 방법

```cpp
plugin::NudgeAgent* nudge = plugin::NudgeAgent::getInstance();
nudge->startSession(API_KEY);

log(nudge->getTestDeviceId().c_str(), "");
```
안드로이드의 경우에 TestDeviceId가 공백으로 return 되는 경우가 있습니다.
이 경우는 내부적으로 Test Device Id를 log로 출력합니다.

2. setPrintTestDeviceId() 메소드를 사용하여 화면에 Device ID를 표시하는 방법

```cs
plugin::NudgeAgent* nudge = plugin::NudgeAgent::getInstance();
nudge->startSession(API_KEY);

nudge->setPrintTestDeviceId(true);
nudge->load();
nudge->show();
```

테스트 디바이스 아이디를 확인한 이후에는, [Dashboard](https://admin.adfresca.com)를 접속하여 'Test Device' 메뉴를 통해 디바이스 등록이 가능합니다.

* * *

## IAP, Reward and Sales Promotion

### In-App Purchase Tracking

_In-App-Purchase Tracking_  기능을 통하여 현재 앱에서 발생하고 있는 모든 인-앱 결제를 분석하고 캠페인 타겟팅에 이용할 수 있습니다.

Nudge의 In-App-Purchase Tracking은 2가지 유형이 있습니다.

1. 실제 화폐를 통해 결제되는 Hard Currency Item Purchase Tracking (예: USD $1.99를 결제하여 Gold 100개 아이템을 구입)
2. 가상 화폐를 통해 결제되는 Soft Currency Item Purchase Tracking (예: Gold 10개를 이용하여 포션 아이템을 구입)

위 2가지 유형의 데이터를 모두 Tracking 함으로써 앱의 매출뿐만 아니라 인-앱 사용자들의 아이템 구매 추이 분석까지 가능합니다.

아이템 정보 등록을 위한 별도의 작업은 필요하지 않으며, 클라이언트에서 결제된 아이템 정보가 자동으로 대쉬보드에 등록되는 방식입니다. (아이템 리스트 확인은 대쉬보드 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.)

아래의 적용 예제를 참고하여 간단히 In-App-Purchase Tracking 기능을 적용합니다.

### Hard Currency Item Tracking

Hard Currency Item의 결제는 각 앱스토어별 인-앱 결제 라이브러리를 통해 이루어집니다. 각 결제 라이브러리에서 _'결제 성공'_ 이벤트가 발생 할 시에 Purchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다. 그리고 _'결제 실패'_ 이벤트가 발생 할 시에는 cancelPromotionPurchase() 메소드를 호출합니다.

적용 예제 1: Cocos2D-X 환경에서 결제 성공 이벤트 발생 시
```cpp
using namespace cocos2d::plugin;
void Scene::onHardItemPurchased() 
{
	std::map<std::string, std::string> receiptMap;
    receiptMap["orderId"] = "google_play_order_ID";
    receiptMap["receiptData"] = "google_play_receipt_data_json";
    receiptMap["signature"] = "google_play_signature";
    
    NudgePurchase* purchase = new NudgePurchase(NudgePurchase::TYPE::TYPE_HARD);
    purchase->setItemId("gold100")
    			->setItemName("100 Gold")
                ->setCurrencyCode("USD")
                ->setPrice(0.99)
                ->setPurchaseDate(purchaseDateTimeString)
                ->setReceipt(receiptMap);
    NudgeAgent::getInstance()->logPurchase(purchase);
}

void Scene::onPurchaseHardItemFailure() 
{
	NudgeAgent::getInstance()->cancelPromotionPurchase();
}
```

Hard Currency Item을 위한 PurchaseBuilder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
setItemId(std::string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. Nudge 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
setItemName(std::string) | 결제한 아이템의 이름을 설정합니다. 이 값은 대쉬보드에서 이름을 표시하기 위한 용도로만 이용됩니다. 등록된 이름은 언제든지 대쉬보드에서 변경이 가능합니다. 
setCurrencyCode(std::string) | ISO 4217 표준 코드를 설정합니다. Google Play의 경우 'Default price' 에 설정되는 Currency Code 값을 이용하며 타 결제 라이브러리의 경우는 보통 이용 가능한 Currency Code가 고정되어 있습니다 (예: 아마존은 USD, 티스토어는 KRW). 또는 자체 백엔드 서버에서 결제하는 아이템의 Currency Code를 내려받아 설정할 수 있습니다.
setPrice(double) | 아이템의 가격을 설정합니다. 결제 라이브러리에서 주는 값을 이용하거나, 자체 백엔드 서버에서 가격을 내려받아 설정할 수 있습니다. 
setPurchaseDate(std::string) | 결제된 시간을 문자열로 설정합니다. 값이 설정되지 않은 경우 Nudge 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.
setReceipt(std::map&lt;std::string, std::string&gt;) | 추후 Receipt Verficiation 기능을 위해 필요한 데이터를 설정합니다. 현재 버전의 SDK는 Google Play만 지원하며 타 결제 라이브러리의 경우는 값을 설정하지 않습니다.

### Soft Currency Item Tracking

Soft Currency Item의 결제는 앱 내의 가상 화폐로 아이템을 결제한 경우를 의미합니다. 앱 내에서 가상 화폐를 이용한 결제 이벤트가 성공한 경우 아래 예제와 같이 Purchase 객체를 생성하고 logPurchase(purchase) 메소드를 호출합니다. 그리고 _'결제 실패'_ 이벤트가 발생 할 시에는 cancelPromotionPurchase() 메소드를 호출합니다.

적용 예제: 
```cpp
using namespace cocos2d::plugin;
void Scene::onSoftItemPurchased()
{
    NudgePurchase* purchase = new NudgePurchase(NudgePurchase::TYPE::TYPE_SOFT);
    purchase->setItemId("long_sword")
    			->setItemName("Long Sword")
                ->setCurrencyCode("gold")
                ->setPrice(100)
                ->setPurchaseDate(purchaseDateTimeString); // std::string
    NudgeAgent::getInstance()->logPurchase(purchase);
}

void Scene::onPurchaseSoftItemFailure()
{
	NudgeAgent::getInstance()->cancelPromotionPurchase();
}
```

Soft Currency Item을 위한 PurchaseBuilder의 보다 자세한 설명은 아래와 같습니다.

Method | Description
------------ | ------------- | ------------
setItemId(std::string) | 결제한 아이템의 고유 식별 아이디를 설정합니다. 등록된 앱스토어에 상관 없이 앱내에서 고유한 식별 값을 이용하는 것을 권장합니다. Nudge 대쉬보드에서 해당 값을 기준으로 아이템 목록이 생성됩니다. 
setItemName(std::string) | 결제한 아이템의 이름을 설정합니다. 이 값은 대쉬보드에서 이름을 표시하기 위한 용도로만 이용됩니다. 등록된 이름은 언제든지 대쉬보드에서 변경이 가능합니다. 
setCurrencyCode(std::string) | 결제에 사용한 가상화폐 고유 코드를 설정합니다. (예: gold)
setPrice(double) | 가상 화폐로 결제한 가격 정보를 설정합니다. (예: gold 10개의 경우 10 값을 설정)
setPurchaseDate(std::string) | 결제된 시간을 문자열 형태로 설정합니다. 값이 설정되지 않은 경우 Nudge 서비스에 기록되는 시간이 결제 시간으로 자동 설정됩니다.

### IAP Trouble Shooting

LogPurchase() 메소드를 통해 기록된 Purchase 객체는 Nudge 서비스에 업데이트되어 실시간으로 대쉬보드에 반영됩니다. 현재까지 등록된 아이템 리스트는 'Overview' 메뉴의 Settings - In App Items 페이지를 통해 확인할 수 있습니다.

만약 아이템 리스트가 새로 갱신되지 않는 경우, Android SDK의 AFPurchaseExceptionListener 구현하여 혹시 에러가 발생하고 있지 않은지 확인해야 합니다. Purchase 객체의 값이 제대로 설정되지 않은 경우, AFPurchaseExceptionListener 통하여 에러 메시지가 표시됩니다.

Unity Plugin의 경우는 이미 아래와 같은 코드가 적용되어 있습니다. 따라서 Purchase 객체가 제대로 생성되지 않은 경우 "AFPurchaseExceptionListener.onException = {error message}" 형태의 로그가 콘솔에 출력됩니다.

```java
......
AdFresca.getInstance(UnityPlayer.currentActivity).logPurchase(purchase, new AFPurchaseExceptionListener(){
  public void onException(AFPurchase purchase, AFException e) {
    Log.e("AdFresca", "AFPurchaseExceptionListener.onException = " + e.getMessage());
  }
});
```

* * *

### Give Reward

Reward 기능을 적용하여 현재 사용자에게 지급 가능한 보상 아이템이 있는지 확인하고, 지정된 보상 아이템을 사용자에게 지급할 수 있습니다.

Reward 캠페인의 'Reward Item' 항목을 설정했거나, Incentivized CPI & CPA 캠페인의 'Incentive Item' 을 설정한 경우 사용자에게 보상 아이템이 지급됩니다.

먼저 AndroidManifest.xml 내용을 확인합니다.

```xml
<manifest>   
  <application>
      .........
      <activity android:name="com.adfresca.sdk.reward.AFRewardActivity" />
      .........
   </application>
</manifest>
```

이제 구현을 위해서 아래 코드를 사용합니다.
- CheckRewardItems(): 현재 지급 가능한 보상 아이템이 있는지 검사합니다. 사용자가 앱을 실행할 호출하는 것을 권장합니다.
- SetAndroidRewardItemListener(): 아이템 지급 조건이 만족되면 onReward 이벤트가 발생하도록 설정할 수 있습니다. 이벤트를 발생시킬 게임 오브젝트와 이벤트 메소드 이름을 지정합니다. Android에 한해서 제공되며 iOS의 경우 아래와 같이 AFRewardItemDelegate를 네이티브 상에서 구현하여 유니티로 이벤트를 넘겨줍니다.

**iOS에서 AFRewardItemDelegate 구현하기**

```objective-c
// UnityAppController.h

@interface UnityAppController : NSObject<UIApplicationDelegate, AFRewardItemDelegate>
{
  ...
}

```

```objective-c
// UnityAppController.mm

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  [[AdFrescaView shardAdView] setRewardDelegate:self];
}

- (void)itemRewarded:(AFRewardItem *)item 
{
  UnitySendMessage("YourGameObject", "OnReward", [[item JSON] UTF8String]);
}

```

**Unity 코드 적용하기**

```cs
void Start ()
{
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetGCMSenderId(GCM_SENDER_ID);    
  plugin.StartSession();

  plugin.SetAndroidRewardItemListener("YourGameObject", "OnReward");
  plugin.CheckRewardItems();
}

public void OnReward(string json)
{
  RewardItem rewardItem = LitJson.JsonMapper.ToObject<RewardItem>(json);
  
  Debug.Log ("rewardItem.name: " + rewardItem.name);
  Debug.Log ("rewardItem.quantity: " + rewardItem.quantity);
  Debug.Log ("rewardItem.uniqueValue: " + rewardItem.uniqueValue);
  Debug.Log ("rewardItem.securityToken: " + rewardItem.securityToken);
  
  SendItemToUser("USER_ID", rewardItem.uniqueValue, rewardItem.quantity, rewardItem.securityToken);
}
```

캠페인 종류에 따라 onReward 이벤트의 발생 조건이 다릅니다.

- Reward 캠페인: 캠페인이 앱 사용자에게 매칭되어 노출될 때 이벤트가 발생합니다
- Incentivized CPI 캠페인: 사용자의 Advertising App 설치가 확인된 후 이벤트가 발생합니다.
- Incentivized CPA 캠페인: 사용자의 Advertising App 설치가 확인되고 보상 조건으로 지정된 마케팅 이벤트가 호출된 후에 발생합니다.

만일 디바이스의 네트워크 단절이 발생한 경우 SDK는 데이터를 로컬에 보관하여 다음 앱 실행에서 아이템 지급이 가능하도록 구현되어 있기 때문에 항상 100% 지급을 보장합니다.

##### SendItemToUser() 메소드의 구현

SDK에서 요청한 아이템을 사용자에게 지급해야 합니다. 클라이언트에서 직접 지급하거나, 백엔드 서버와 통신하여 사용자의 선물함으로 보상을 지급하는 방식을 이용할 수 있습니다. 아이팀의 고유 값, 수량, 그리고 시큐리티 토큰값을 이용하여 현재 로그인된 사용자에게 아이템을 지급합니다.

##### 백엔드 서버를 통해 아이템을 지급할 경우 보안 이슈 해결하기

저희 SDK에서는 특정 사용자가 동일한 캠페인에서 1회 이상 아이템이 지급되지 않도록 처리하고 있습니다. 하지만 클라이언트에서 백엔드 서버와 통신하는 과정이 노출된다면 외부 공격에 의해 아이템이 중복으로 지급되는 보안 이슈가 발생할 수도 있습니다. 해당 문제를 방지하기 위하여 시큐리티 토큰값을 제공하고 있습니다. 시큐리티 토큰값은 대쉬보드에서 캠페인 생성 시에 자동으로 생성 혹은 직접 지정할 수 있는 고유 값입니다. 해당 값을 이용하여 아래와 같은 처리를 할 수 있습니다.

1. 백엔드 서버에서는 진행하려는 리워드 캠페인의 시큐리티 토큰값을 데이터베이스에 미리 보관하여, 존재하지 않는 토큰값이 포함된 요청을 거절합니다.
2. 특정 사용자가 동일한 토큰값으로 1회 이상 지급 요청을 하는 경우 요청을 거절합니다. 
3. 만약 토큰값이 외부에 노출되었다고 판단될 경우, 대쉬보드에서 토큰값을 새로 생성하거나 수정합니다.

### Sales Promotion

Sales Promotion 캠페인을 이용하여 특정 아이템의 구매를 유도할 수 있습니다. 사용자가 캠페인에 노출된 이미지 메시지를 클릭할 경우 해당 아이템의 결제 UI가 표시됩니다. SDK는 사용자의 실제 결제 여부까지 자동으로 트랙킹하여 대쉬보드에서 실시간으로 통계를 제공합니다. 

프로모션 기능을 적용하기 위해서 OnPromotion 이벤트를 구현합니다. 프로모션 캠페인이 노출된 후 사용자가 이미지 메시지의 액션 영역을 탭하면 onPromotion() 이벤트가 발생합니다. 이벤트에 넘어오는 PromotionPurchase 객체 정보를 이용하여 사용자에게 아이템 결제 UI를 표시하도록 코드를 적용합니다.

Hard Currency 아이템의 경우 인-앱 결제 라이브러리를 이용하여 결제 UI를 표시합니다. PromotionPurchase 객체의 ItemId 값이 아이템의 SKU 값에 해당됩니다. 

Soft Currency 아이템의 경우는 앱이 기존에 사용하고 있는 상점 내 아이템 결제 UI를 표시하도록 코드를 작성합니다. Soft Currency 프로모션의 경우는 2가지 가격 할인 옵션을 제공하고 있습니다. PromotionDiscountType 값을 이용하여 할인 옵션을 확인할 수 있습니다.

1. **Discount Price**: 캠페인에 직접 지정된 가격으로 아이템을 판매합니다. Price 값을 이용하여 가격 정보를 얻습니다.
2. **Discount Rate**: 캠페인에 지정된 할인율을 적용하여 아이템을 판매합니다. PromotionDiscountRate 값을 이용하여 할인율 정보를 받아옵니다.

**iOS에서 AFPromotionDelegate 구현하기**

```objective-c
// UnityAppController.h
@interface UnityAppController : NSObject<UIApplicationDelegate, AFPromotionDelegate>
{
  ...
}

...

// UnityAppController.mm
- (void)applicationDidBecomeActive:(UIApplication *)application 
{
  AdFrescaView *fresca = [AdFrescaView sharedAdView];
  [fresca setPromotionDelegate:self];
}

- (void)onPromotion:(AFPurchase *)promotionPurchase
{
  UnitySendMessage("YourGameObject", "OnPromotion", [[promotionPurchase JSONForUnity] UTF8String]);
}
```

**Unity 코드 적용하기**

```cs
void Start ()
{
	AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
	plugin.Init(API_KEY);
	plugin.StartSession();
	
	....
	
	plugin.SetPromotionListener("YourGameObject", "OnPromotion"); // for Android only
}
	
public void OnPromotion(string json)
{
	Debug.Log("OnPromotion = " + json);		
	Purchase PromotionPurchase = LitJson.JsonMapper.ToObject<Purchase>(json);

	string ItemId = PromotionPurchase.ItemId;
	string LogMessage = "no logMessage";

	if (PromotionPurchase.PurchaseType == Purchase.Type.HARD_ITEM)
	{
		// 인-앱 결제 라이브러리를 호출.
		ShowHardItemPurchaseUI(ItemId);
		LogMessage = String.Format("on HARD_ITEM Promotion ({0})", ItemId);  
	}
	else if (PromotionPurchase.PurchaseType == Purchase.Type.SOFT_ITEM)
	{
		String CurrencyCode = PromotionPurchase.CurrencyCode;
		if (PromotionPurchase.PromotionDiscountType == Purchase.DiscountType.DISCOUNTED_TYPE_PRICE) 
		{
      // 할인된 가격을 이용하여 앱-내 아이템 구매 UI를 표시.
			double DiscountedPrice = PromotionPurchase.Price;
			ShowSoftItemPurchaseUIWithDiscountedPrice(ItemId, CurrencyCode, DiscountedPrice);
			LogMessage = String.Format("on SOFT_ITEM Promotion ({0}) with {1} {2}", ItemId, DiscountedPrice, CurrencyCode);
		}
		else if (PromotionPurchase.PromotionDiscountType == Purchase.DiscountType.DISCOUNT_TYPE_RATE)
		{
      // 할인율 값을 이용하여 앱-내 아이템 구매 UI를 표시. discountedPrice = originalPrice - (originalPrice * discountRate)
			double DiscountRate = PromotionPurchase.PromotionDiscountRate;
			ShowSoftItemPurchaseUIWithDiscountRate(ItemId, CurrencyCode, DiscountRate);
			LogMessage = String.Format("on SOFT_ITEM Promotion ({0}) with {1}% discount", ItemId, DiscountRate * 100.0);
		}
	}

	Debug.Log(LogMessage);
}
```

SDK가 사용자의 실제 구매 여부를 트랙킹하기 위해서는 [In-App Purchase Tracking](#in-app-purchase-tracking) 기능이 미리 구현되어 있어야 합니다. 특히 사용자가 아이템을 구매를 하지 않거나 실패한 경우를 트랙킹 하기 위하여 CancelPromotionPurchase() 메소드가 반드시 적용되어 있어야 합니다.

* * *

## Dynamic Targeting

### Custom Parameter

커스텀 파라미터는 캠페인 진행 시, 타겟팅을 위해 사용할 사용자의 상태 값을 의미합니다.

Nudge SDK는 기본적으로 '국가, 언어, 앱 버전, 실행 횟수 등'의 디바이스 고유 데이터를 수집하며, 동시에 각 앱 내에서 고유하게 사용되는 특수한 상태 값들(예: 캐릭터 레벨, 보유 포인트, 스테이지 등)을 커스텀 파라미터로 정의하고 수집하여 분석 및 타겟팅 기능을 제공합니다.

커스텀 파라미터 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Custom Parameters 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 커스텀 파라미터의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

Integer, Boolean 형태의 데이터를 상태 값으로 설정할 수 있으며, *SetCustomParameter** 메소드를 사용하여 각 인덱스 값에 맞게 상태 값을 설정합니다. 앱이 실행되는 시점에 한 번 설정하고, 해당 파라미터 값이 변경될 때 마다 최신 값을 설정하여 줍니다.

```cs
void Start() {
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Init(API_KEY);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, User.level);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_AGE, User.age);
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_HAS_FB_ACCOUNT, User.hasFacebookAccount);
  plugin.StartSession();
}

  .....

void onUserLevelChanged(int level) {
  User.level = level
  
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, User.level);
}
```

* * *

### Marketing Moment

마케팅 모멘트는 유저에게 메세지를 전달하고자 하는 상황을 의미합니다. (예: 캐릭터 레벨 업, 퀘스트 달성, 스토어 페이지 진입)

마케팅 모멘트 기능을 사용하여 지정된 상황에 알맞는 캠페인이 노출되도록 할 수 있습니다.

마케팅 모멘트 설정은 [Dashboard](https://admin.adfresca.com) 사이트를 접속하여 앱의 Overview 메뉴 -> Settings - Marketing Moments 버튼을 클릭하여 확인할 수 있습니다.

SDK 적용을 위해서는 Dashboard에서 지정된 각 마케팅 모멘트의 '인덱스' 값이 필요합니다. 인덱스 값은 1,2,3,4 와 같은 Integer 형태의 고유 값이며 소스코드에 Constant 형태로 지정하여 이용하는 것을 권장합니다.

각 모멘트 발생 시, Load() 메소드에 원하는 모멘트 인덱스 값을 인자로 넘겨주시면 간단히 적용이 완료됩니다.

(Load() 메소드에 인덱스를 설정하지 않은 경우, 인덱스 값은 '1' 값이 자동으로 지정됩니다.)

**Example**:  사용자가 메인 페이지로 이동할 시에 설정한 컨텐츠를 노출

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.Load(EVENT_INDEX_MAIN_PAGE);  // 메인 페이지에 설정한  컨텐츠를 노출
plugin.Show();
```

**Example**: 사용자의 게임 캐릭터가 레벨업을 했을 때 설정한 컨텐츠를 노출

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetCustomParameter(CUSTOM_PARAM_INDEX_LEVEL, level); // 사용자 level 정보를 가장 최신으로 업데이트
plugin.Load(EVENT_INDEX_LEVEL_UP); // 레벨업 이벤트에 설정한 컨텐츠를 노출
plugin.Show();
```

## Advanced

### Timeout Interval

메시지의 최대 로딩 시간을 직접 지정하실 수 있습니다. 지정된 시간 내에 메시지가 로딩되지 못한 경우, 사용자에게 메시지를 표시하지 않습니다.

최소 1초 이상 지정이 가능하며, 지정하지 않을 시 기본 값으로 5초가 지정 됩니다.

```cs
AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
plugin.SetTimeoutInterval(5);
plugin.Load();
plugin.Show();
```

* * *

## Reference

### Deep Link

캠페인의 Deep Link 설정 시에 Custom URL Schema를 지정할 수 있습니다. 이를 통해 사용자가 콘텐츠를 클릭할 경우, 자신이 원하는 특정 앱 페이지로 이동하는 등의 액션을 지정할 수 있습니다.

#### Android 환경에서 Custom URL 적용하기

네이티브 애플리케이션 개발 환경에서는 AndroidManifest.xml 파일을 수정하여 원하는 액티비티에 scheme 정보를 추가하는 방식으로 적용이 됩니다.

하지만 액티비티를 페이지 개념으로 사용하는 네이티브 환경과 달리, Unity 엔진을 사용하여 안드로이드 애플리케이션을 개발하는 경우 단 하나의 UnityPlayer 액티비티만을 사용하며 엔진 내부적으로 페이지를 처리합니다.

때문에 schema를 지정할 수 있는 액티비티의 제약이 생깁니다. MAIN 으로 지정된 UnityPlayer 액티비티는 url schema를 적용할 수 없습니다. 그래서 아래와 같은 방법들을 사용하여 Custom URL을 처리합니다.

1) UnityPlayer 액티비티의 startActivity(intent) 메소드를 오버라이딩하여 Custom URL 처리하기 (인-앱 메시징 캠페인)

인-앱 메시징 캠페인을 통해 전달되는 Deep Link은 항상 인게임 상황에서 전달되며, SDK가 내부적으로 startActivity() 메소드를 이용하여 호출하고 있습니다. 이러한 조건에서는 게임이 실행되고 있는 UnityPlyaer 액티비티의 startActivity() 메소드를 직접 구현함으로써 Custom URL 처리가 가능합니다.

먼저 Eclipse에서 Android Project를 생성하여 UnityPlayerActivity를 상속받은 'Main Actvity' 클래스를 생성합니다. 그리고 AndroidMenefest.xml 파일을 아래와 같이 수정합니다.

```xml
<application android:icon="@drawable/app_icon" android:label="@string/app_name" android:debuggable="true">
  <activity android:name="com.MyCompany.ProductName.MainActivity" android:label="@string/app_name">
    <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
  </activity>
  ..... 
</application>
```
startActivity() 메소드를 아래와 같이 구현합니다. 'myapp://' 으로 적용된 Custom URL이 전달된다면 새로 액티비티를 호출하지 않고 미리 설정한 게임 오브젝트로 값을 전달합니다.

```java
public class MainActivity extends UnityPlayerActivity {
  ...
  @Override 
  public void startActivity(Intent intent) { 
    boolean isStartActivity = true;

    // Check intent 
    Uri uri = intent.getData(); 
    if (uri != null && uri.getScheme().equals("myapp")) { 
      isStartActivity = false; 
    }

    if (isStartActivity) { 
      super.startActivity(intent); 
    } else { 
      // do something with UnitySendMessage and uri 
      Log.d("TEST", "MainActivity.startActivity() : uri = " + uri.toString());    
      UnityPlayer.UnitySendMessage("Fresca", "OnCustomURL", uri.toString());
    } 
  }
}
```

2) Push Notification을 통해 넘어오는 Custom URL 처리하기 (푸시 메시징 캠페인)

Custom URL이 설정된 Push Notification을 수신한 경우, Notification을 터치 시 원하는 액션을 지정할 수 있습니다. 단, 이 경우는 인게임 상황이 아니기 때문에 조금 다른 방법을 사용합니다.

먼저 PushProxyActivity 라는 이름의 액티비티 클래스를 하나 생성합니다. 그리고 AndroidMenefest.xml 내용을 아래와 같이 추가합니다. 

```xml
<activity android:name=".PushProxyActivity">
  <intent-filter> 
    <action android:name="android.intent.action.VIEW" /> 
    <category android:name="android.intent.category.DEFAULT" /> 
    <category android:name="android.intent.category.BROWSABLE" /> 
    <data android:scheme="myapp" android:host="com.adfresca.push" />
  </intent-filter> 
</activity>

.......
```
위와 같이 설정한 경우 푸시 메시징 캠페인에서는 myapp://com.adfresca.push?item=abc 와 같은 형식의 url을 입력해야 합니다.

다음은 PushProxyActivity 클래스의 내용을 구현해야 합니다. PushProxyActivity 클래스는 Android OS로 부터 수신하는 Custom URL 정보를 받아 처리하고 바로 자신을 종료하는 단순한 프록시 형태의 액티비티입니다. 만약 현재 UnityPlayer가 실행 중이 아니라면 Custom URL을 처리할 수 없으므로 새로 UnityPlayer를 실행하여 uri 값을 넘겨야 합니다.

```java
public class PushProxyActivity extends Activity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    
    // hide ui
    requestWindowFeature(Window.FEATURE_NO_TITLE);
    getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);
    getWindow().setBackgroundDrawableResource(android.R.color.transparent);
            
    Uri uri = getIntent().getData();
    if (UnityPlayer.currentActivity != null) { 
      Log.d("TEST", "PushProxyActivity.onCreate() with currentActivity : uri = " + uri.toString());   
      UnityPlayer.UnitySendMessage("Fresca", "OnCustomURL", uri.toString()); 
      
     } else {
       Log.d("TEST", "PushProxyActivity.onCreate() without currentActivity : uri = " + uri.toString());   
       
       // Start a new player with uri
       try {
         Intent intent = new Intent(this, MainActivity.class);
         intent.putExtra("fresca_uri", uri.toString());
         intent.setFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP | Intent.FLAG_ACTIVITY_SINGLE_TOP);
         startActivity(intent);
       } catch (Exception e) {
         e.printStackTrace();
       }
    }
    
    finish();
  }
}
```
마지막으로 PushProxyActivity를 통해 게임이 실행된 경우 넘어오는 uri 값을 처리합니다. Main 액티비티에 아래와 같은 내용을 추가합니다.

```java
public class MainActivity extends UnityPlayerActivity {
  @Override
  public void onCreate(Bundle savedInstanceState) {
    ......
    // Handle custom uri from PushProxcyActivity
    String frescaURL = this.getIntent().getStringExtra("fresca_uri");
    if (frescaURL != null) {
      Log.d("TEST", "MainActivity.onCreate() with uri");  
      UnityPlayer.UnitySendMessage("Fresca", "OnCustomURL", frescaURI);
    }   
    ......
  }
  .........
}
```

Android 플랫폼 환경에서 Custom URL을 처리할 수 있는 모든 방법을 구현하였습니다.

#### iOS 환경에서 Custom URL 적용하기

iOS의 경우 1개의 이벤트체서 모든 URL 처리가 가능하기 때문에 비교적 간단하게 적용할 수 있습니다.

1) Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

<img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png" />

2) AppController.mm 파일을 열어 handleOpenURL 메소드를 구현합니다. 호출되는 URL 값을 유니티 게임 오브젝트에 전달합니다. 

```mm
- (BOOL)application:(UIApplication *)application handleOpenURL:(NSURL *)url 
{  
  if ([url.scheme isEqualToString:@"myapp"])
  {
        NSString *frescaURL = [url absoluteString];
        UnitySendMessage("Fresca", "OnCustomURL", [frescaURL UTF8String]);
  }
  return YES;
}
```

#### Unity에서 url 값 확인 및 처리하기

위 예제에서는 모두 'Fresca' 게임 오브젝트의 'OnCustomURL' 이벤트 메소드를 호출하여 값을 전달하였습니다. 이제 Unity 게임 상에서 그 값을 직접 받아 확인하고 처리할 수 있습니다.

```java
public void OnCustomURL(string url)
{
  Debug.Log("OnCustomURL = " + url); 
  //ex) myapp://com.adfresca.custom?item=abc 값이 전달된 경우 item=abc 값을 파싱하여 아이템 지급
}
```

* * *

### Cross Promotion Configuration

Incentivized CPI & CPA 캠페인 기능을 사용하여, 사용자가 Media App에서 Advertising App의 광고를 보고 앱을 설치하였을 때 보상으로 Media App의 아이템을 지급할 수 있습니다.

- Medial App: 다른 앱의 광고를 노출하고, 광고 대상의 앱을 설치한 사용자들에게 보상을 지급하는 앱
- Advertising: Media App에 광고가 노출되는 앱.

Incentivized CPI & CPA 캠페인에 대한 보다 자세한 설명 및 [Dashboard](https://admin.adfresca.com) 사이트에서의 설정 방법은 [크로스 프로모션 캠페인 이해하기](https://adfresca.zendesk.com/entries/22033960) 가이드를 참고하여 주시기 바랍니다.

SDK 적용을 위해서는 Advertising App에서의 URL Schema 설정 및 Media App에서의 Reward Item 지급 기능을 구현해야 합니다.

#### Advertising App 설정하기:
  1. Android

  Android 플랫폼의 경우 앱의 패키지 이름을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 패키지 이름을 확인하고 CPI Identifier로 사용합니다.

  AndroidManifest.xml 파일을 열어 패키지 이름을 확인합니다.

  ```xml
  <?xml version="1.0" encoding="utf-8"?>

  <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="com.adfresca.demo">
    <application>
    .....
    </application>
  </manifest>
  ```

  위 경우 [Dashboard](https://admin.adfresca.com) 사이트에서 Advertising App의 CPI Identifier 값을 'com.adfresca.demo' 으로 설정하게 됩니다. 

  2. iOS

  iOS 플랫폼의 경우 URL Schema 값을 이용하여 광고를 노출한 앱이 실제로 디바이스에 설치되었는지 검사하게 됩니다. 따라서 Advertising App 앱의 URL Schema을 설정하고 CPI Identifier로 사용합니다.

  Xcode 프로젝트의 Info.plst 파일을 열어 사용할 URL Schema 정보를 설정 합니다.

  <img src="https://adfresca.zendesk.com/attachments/token/n3nvdacyizyzvu0/?name=Screen+Shot+2013-02-07+at+6.51.09+PM.png"/>

  위 경우 [Dashboard](https://admin.adfresca.com) 사이트에서 Advertising App의 CPI Identifier 값을 'myapp://' 으로 설정하게 됩니다. 
  iOS 플랫폼의 경우 URL Schema 값이 다른 앱과 중복될 수 있습니다. 정상적인 캠페인 진행을 위해서는 최대한 Unique한 값을 선택해야 합니다.
  
  마지막으로, Incentivized CPI 캠페인을 진행할 경우, Advertising App의 SDK 설치는 필수가 아니며 CPI Identifier 설정만 진행되면 됩니다. 하지만 Incentivized CPA 캠페인을 진행할 경우 반드시 SDK 설치가 필요하며 보상 조건으로 지정한 마케팅 모멘트르 발생되어야 합니다. 사용자가 보상 조건을 완료한 이후 아래와 같이 유니티 코드로 지정한 마케팅 모멘트 호출합니다.

  ```cs
  // 튜토리얼 완료 이벤트를 보상 조건으로 지정한 경우
  AdFresca.Plugin plugin = AdFresca.Plugin.Instance;
  plugin.Load(EVENT_INDEX_TUTORIAL);  
  plugin.Show(EVENT_INDEX_TUTORIAL);
  ```

#### Media App SDK 적용하기:

  Media App에서 보상 지급 여부를 확인하고, 사용자에게 아이템을 지급하기 위해서는 SDK 가이드의 [Give Reward](#give-reward) 항목의 내용을 구현합니다.

* * *

### Image Push Notification

Image Push Notification 기능 적용에 대한 내용은 Android SDK 가이드의 ["Image Push Notification"](https://github.com/nudge-now/sdk-android/blob/master/README.kor.md#image-push-notification) 내용을 참고하여 진행이 가능합니다.

* * *

### Proguard Configuration

안드로이드에서 Proguard 툴을 이용하여 APK 파일을 보호하는 경우 몇 가지 예외 처리 작업을 진행해야 합니다. Nudge SDK와 SDK에 포함된 OpenUDID 및 Google Gson에 대한 예외 처리를 아래와 같이 적용합니다.

```java
-keep class com.adfresca.** {*;} 
-keep class com.google.gson.** {*;} 
-keep class org.openudid.** {*;} 
-keep class sun.misc.Unsafe { *; }
-keepattributes Signature 
```

* * *

## Troubleshooting

콘텐츠가 화면에 제대로 출력되지 않거나, 에러가 발생하는 경우 SDK에서 에러 정보를 확인할 수 있습니다. 현재 Unity 코드로 에러 정보를 출력하는 방법은 아직 지원되지 않고 있으며, 각 플랫폼 코드에서 직접 로그를 출력할 수 있습니다.

**Android**

UnityPlayerActivity 클래스를 오버라이드해서 사용하고 있거나, 다른 자바 플러그인을 이용하는 있는 경우 아래 코드를 적용하여 로그를 출력할 수 있습니다. (혹은 UnitySendMessage 메소드를 이용하여 유니티로 이벤트를 전달할 수 있습니다.)

```java
AdFresca.setExceptionListener(new AFExceptionListener(){
  @Override
  public void onExceptionCaught(AFException e) {
    Log.w("TAG", e.getCode() + ":" + e.getLocalizedMessage());
  }
});
```

**iOS**

Xcode 프로젝트에서 AdFrescaViewDelegate를 구현하여 로그를 출력할 수 있습니다. (혹은 UnitySendMessage 메소드를 이용하여 유니티로 이벤트를 전달할 수 있습니다.)

```objective-c
// UnityAppController.h
@interface UnityAppController : NSObject<UIApplicationDelegate, AdFrescaViewDelegate>
{
  .....
}
```

```objective-c
// UnityAppController.mm

- (BOOL)application:(UIApplication*)application didFinishLaunchingWithOptions:(NSDictionary*)launchOptions
{
  [AdFrescaView startSession:@"YOUR_API_KEY"];
  AdFrescaView *view = [AdFrescaView shardAdView];
  view.delegate = self;
}

- (void)fresca:(AdFrescaView *)fresca didFailToReceiveAdWithException:(AdException *)error {  
  NSLog(@"AdException message : %@", [error message]);
}
```

* * *

## Release Notes
- **v2.2.3 _(2014/12/05 Updated)_**
  - Purchase 객체에 HARD_ITEM, SOFT_ITEM purchase type이 추가되고 ACTUAL_ITEM, SOFT_ITEM 값이 deprecated 되었습니다. 자세한 내용은 [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요.
- **v2.2.3 (2014/11/11 Updated)**
    - iOS 플랫폼에서의 [In-App Purchase Tracking](#in-app-purchase-tracking) 기능을 지원합니다.
    - iOS 플랫폼에서의 [Sales Promotion](#sales-promotion) 기능을 지원합니다.
    - 안드로이드 플랫폼에서 GCM Push Registration ID를 직접 지정할 수 있는 SetGCMPushRegistrationIdentifier() 메소드가 추가되었습니다.   
    - [iOS SDK 1.4.8](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- **v2.2.2 (2014/09/30 Updated)**
    - iOS 플랫폼의 A/B 테스트 기능을 지원합니다. 해당 기능은 별도의 코딩 작업 없이 이용 가능합니다.
    - [Android SDK 2.4.4](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.4.6](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.1 _(8/15/2014 Updated)_
    - 리워드 지급 시에 시큐리티 토큰값을 이용하여 보안 이슈를 해결할 수 있습니다. 자세한 내용은 [Give Reward](#give-reward) 항목을 참고하여 주세요.
    - Sales Promotion 캠페인 기능을 이용하여 아이템의 프로모션 기능을 지원합니다. 자세한 내용은 [Sales Promotion](#sales-promotion) 항목을 참고하여 주세요.
    - [In-App Purchase Tracking](#in-app-purchase-tracking) 기능에서 cancelPromotionPurchase() 메소드가 추가되었습니다. 
    - 이미지 메시지의 **Tap Area** 기능을 지원합니다.
    - Android SDK가 캠페인 매칭 시에 여러 개의 캠페인이 동시에 매칭될 수 있도록 지원합니다.. 새로운 SDK는 순차적으로 매칭된 캠페인들의 메시지를 표시합니다.
    - iap beta 버전이 2.2.1부터 통합되었습니다. 
    - [Android SDK 2.4.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta3 _(4/6/2014 Updated)_
    - iOS SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [iOS SDK - Installation](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#installation) 항목을 참고하여 주세요.
    - v2.1.8에서 적용된 '인-앱 메시징 캠페인을 통한 Reward Item 지급 기능'을 지원합니다.
    - v2.1.8에서 적용된 Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요
    - v2.1.8에서 개선된 [Reward Item](#reward-item) 기능이 적용되었습니다. 
    - [Android SDK 2.4.0-beta4](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.3.5](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta2 _(1/31/2014 Updated)_ 
    - [Android SDK 2.4.03](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.2.0-beta1 _(1/14/2014 Updated)_ 
    - 앱 내에서 발생하는 In-App Purchase 데이터를 트랙킹할 수 있는 기능이 추가되었습니다. 자세한 내용은 [In-App Purchase Tracking](#in-app-purchase-tracking) 항목을 참고하여 주세요.
    - [Android SDK 2.4.02](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.8 _(4/6/2014 Updated)_
   - iOS SDK 설치 과정에서 AdSupport framework 추가가 필수항목에서 제외됩니다. IFA 수집을 하지 않아도 SDK 이용이 가능하도록 수정되었습니다. 보다 자세한 내용은 [iOS SDK - Installation](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#installation) 항목을 참고하여 주세요.
   - 인-앱 메시징 캠페인을 통한 Reward Item 지급 기능을 지원합니다.
   - Incentivized CPA 캠페인 기능을 지원합니다. 자세한 내용은 [CPI Identifier](#cpi-identifier) 항목을 참고하여 주세요
   - SetAndroidRewardItemListener 구현 기능이 추가되어, 지급 가능한 아이템이 발생할 시에 자동으로 이벤트가 발생합니다. 보다 자세한 내용은 [Reward Item](#reward-item) 항목을 참고하여 주세요.
    - [Android SDK 2.3.4](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - [iOS SDK 1.3.5](https://github.com/adfresca/sdk-ios/edit/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.7 _(1/31/2014 Updated)_
    - [Android SDK 2.3.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.6 _(1/10/2014 Updated)_ 
    - [Android SDK 2.3.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - Unity 4.3.x for Android 버전에서 ForwardNativeEventsToDalvik 옵션이 설정되지 않은 경우 터치 이벤트가 동작하지 않습니다. 이를 해결하기 위한 자세한 적용 방법은 [Installation](#installation) 항목을 참고하여 주세요.
- v2.1.5 _(12/01/2013 Updated)_ 
    - [iOS SDK 1.3.4](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
- v2.1.4 _(11/27/2013 Updated)_ 
    - [iOS SDK 1.3.3](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
    - [Android SDK 2.3.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.3 _(10/01/2013 Updated)_ 
    - [Android SDK 2.2.3](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.2 _(08/19/2013 Updated)_ 
    - [iOS SDK 1.3.2](https://adfresca.zendesk.com/entries/21346861#release-notes) 버전을 지원합니다.
- v2.1.1
    - [Android SDK 2.2.2](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
- v2.1.0 _(08/08/2013 Updated)_
    - [Android SDK 2.2.1](https://github.com/adfresca/sdk-android-sample/blob/master/README.kor.md#release-notes) 버전을 지원합니다.
    - Android Platform 에서는 TestDeviceId() 메소드 대신 PrintTestDeviceIdByLog() 메소드를 사용하여 연결된 디바이스의 아이디를 확인하도록 변경 되었습니다.
- v2.0.1 _(07/26/2013 Updated)_
    - Plugin에 포함된 GCMIntentService 클래스를 이용하는 경우, 앱이 완전히 종료된 상황에서 푸시 메시지 수신 시 에러 메시지가 발생하는 버그를 수정하였습니다.
    - AndroidPlugin.cs 파일의 기본 매개변수 설정을 삭제하였습니다. 
    - 포함된 Android SDK를 2.1.3 버전으로 업데이트하였습니다.
- v2.0.0 _(07/10/2013 Updated)_
    - _Incentivized CPI_캠페인을 위한 API 가 추가되었습니다.
