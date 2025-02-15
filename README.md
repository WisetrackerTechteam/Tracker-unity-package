## 1. 유니티 플러그인 설치 (AOS/IOS 공통 설정)

### 1.1 유니티 패키지 다운로드
유니티 플러그인 패키지 Wisetracker.unitypackage 파일을 다운로드 해주세요.

### 1.2 유니티 패키지 임포트
Unity Tools에서 Assets > Import Package > Custom Package 메뉴 선택.

다운로드 받은 **Wisetracker.unitypackage** 파일을 선택해주세요.

<img src="http://www.wisetracker.co.kr/wp-content/uploads/2019/08/unity_menu.png" width="340" height="430"/>

![](http://www.wisetracker.co.kr/wp-content/uploads/2019/08/unity_file.png)

### 1.3 Wisetracker AppKey 발급

http://report.wisetracker.co.kr 로그인

설정 > 기본설정 > 서비스 > Android 분석코드 (AppKey) 클릭 > AppKey 복사 후 적용

![Appkey 등록](https://dzf8vqv24eqhg.cloudfront.net/userfiles/6274/8379/ckfinder/images/016.png?dc=201702100857-66 "Appkey 등록")

## 2. 유니티 안드로이드 설정

### AndroidManifest.xml 설정 
-> Assets/Plugins/Android/AndroidManiest.xml

#### 1) Wisetracker AppKey 설정

```xml
<!-- 발급 받은 AppKey meta-data 추가 -->
<meta-data
	android:name="WiseTrackerKey"
	android:value="발급 받은 앱키 추가" />
```

#### 2) 디버깅 모드 설정

```xml
<!-- 개발용 true / 배포용 false 권장 -->
<meta-data
	android:name="WiseTrackerLogState"
	android:value="true" /> 
```

#### 3) 딥링크 설정

딥링크로 진입할 **android:scheme="YOUR_SCHEME"** 스키마와 **android:host="YOUR_HOST"** 호스트를 설정해 주세요. 
**유니티 플러그인에서는 딥링크 진입 동작을 위한 UnityDeepLink를 기본적으로 사용하고 있습니다.**
              
```xml
<!-- 예시는 wisetracker://wisetracker.co.kr 링크로 진입시 딥링크 분석이 가능합니다. -->
<activity android:name="kr.co.wisetracker.UnityDeepLink" 
          android:launchMode="singleTop" >
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:host="wisetracker.co.kr"
              android:scheme="wisetracker" />
    </intent-filter>
</activity>
```

 
## 3. iOS 설정

#### 1) 필수 파일 iosGetFkey.html 추가
Plugin 설치 후 생성된 **해당 프로젝트 폴더/Assets/Plugins/iOS/iosGetFkey.html**  파일을 xCode 내
**Targets > Build Phase > Copy Bundle Resources** 메뉴에 아래와 같이 추가합니다.

![](http://www.wisetracker.co.kr/wp-content/uploads/2020/01/unity_iosGetFkey.png)

#### 2) info.plist파일 디버깅 모드 세팅
info.plist 파일을 open할때 list로 보기 가 아니라 source로 보기를 선탁하신뒤, 아래의 key/value값을 붙혀 넣습니다.

```xml
	// 개발용 true. 배포용 false 권장.
    <key>WiseTrackerLogState</key>
    <string>true</string>
```

#### 3) 외부 유입 경로 분석 ( Deeplink )
앱이 설치된 이후 DeepLink를 통해서 앱이 실행되는 경로 분석이 필요한 경우 
UnityAppController.mm 클래스에 정의된 AppDelegate 정의 항목중 openURL 함수 구현부에 아래와 같이 추가해줍니다.

```Objective-C
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {

    [WiseTracker urlRefererCheck:@"" url:url];
     return YES;
}
```

### 4. 초기화
유니티 앱 실행시 최초 실행되는 MonoBehavior 상속받아 구현된 MainScene 클래스의 Awake() 함수에 다음과 같은 초기화 코드를 삽입해 주세요.

(1) 초기화 호출

```c#
void Awake() 
{
    #if UNITY_ANDROID && !UNITY_EDITOR
        // for android
        WiseTracker.init();
    #elif UNITY_IOS && !UNITY_EDITOR 
        // for ios
        WiseTracker.initialization("제공받은 앱키");
    #endif
}
```

(2) 체류 시간 분석

```c#
void Start() 
{
    // 씬 시작시
    // 체류 시간 분석을 위한 시간(분 단위)을 넣어주세요
    // 예) 15분 단위로 체류 페이지 자동 전송
    WiseTracker.onPlayStart(15);	
}
```

```c#
void OnDestroy() 
{
    // 씬 종료시
    WiseTracker.onPlayStop();
}
```

```c#
void OnApplicationPause(bool pauseStatus)
{
    if (pauseStatus)
    {
    	// 백그라운드 진입시
	WiseTracker.onPlayStop();
    }
}
```
