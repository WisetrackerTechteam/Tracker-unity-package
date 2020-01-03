### 1. 유니티 플러그인 설치 (AOS/IOS 공통 설정)

#### 1.1 유니티 패키지 다운로드
[유니티 플러그인 패키지](https://github.com/WisetrackerTechteam/Tracker-unity-package) 다운로드 해주세요.

#### 1.2 유니티 패키지 임포트
Unity Tools에서 Assets > Import Package > Custom Package 메뉴 선택.

다운로드 받은 **Wisetracker.unitypackage** 파일을 선택해주세요.

![](http://www.wisetracker.co.kr/wp-content/uploads/2019/08/unity_menu.png)
![](http://www.wisetracker.co.kr/wp-content/uploads/2019/08/unity_file.png)

#### 1.3 Wisetracker AppKey 발급

http://report.wisetracker.co.kr 로그인

설정 > 기본설정 > 서비스 > Android 분석코드 (AppKey) 클릭 > AppKey 복사 후 적용

![Appkey 등록](https://dzf8vqv24eqhg.cloudfront.net/userfiles/6274/8379/ckfinder/images/016.png?dc=201702100857-66 "Appkey 등록")

### 2. 유니티 안드로이드 설정

####  AndroidManifest.xml 설정 (Assets/Plugins/Android/AndroidManiest.xml)

##### a) Wisetracker AppKey 설정

```xml
// 발급 받은 AppKey meta-data 추가
<meta-data
	android:name="WiseTrackerKey"
	android:value="발급 받은 앱키 추가" />
```

##### b) 디버깅 모드 설정

```xml
// 개발용 true. 배포용 false 권장.
<meta-data
	android:name="WiseTrackerLogState"
	android:value="true" /> 
```

##### c) 딥링크 설정

딥링크로 진입할 **android:scheme="YOUR_SCHEME"** 스키마와 **android:host="YOUR_HOST"** 호스트를 설정해 주세요. 
**유니티 플러그인에서는 딥링크 진입 동작을 위한 UnityDeepLink를 기본적으로 사용하고 있습니다.**
              
```xml
// 예시는 wisetracker://wisetracker.co.kr 링크로 진입시 딥링크 분석이 가능합니다.
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

 
### 3. iOS 설정

#### 1) info.plist파일 디버깅 모드 세팅
info.plist 파일을 open할때 list로 보기 가 아니라 source로 보기를 선탁하신뒤, 아래의 key/value값을 붙혀 넣습니다.
// 개발용 true. 배포용 false 권장.
```xml
    <key>WiseTrackerLogState</key>
    <string>true</string>
```

#### b) 외부 유입 경로 분석 ( Deeplink )
앱이 설치된 이후 DeepLink를 통해서 앱이 실행되는 경로 분석이 필요한 경우 
UnityAppController.mm 클래스에 정의된 AppDelegate 정의 항목중 openURL 함수 구현부에 아래와 같이 추가해줍니다.
```Objective-c
    [WiseTracker applicationKey:@"제공된 앱키"];
    [WiseTracker setApplication:[UIApplication sharedApplication]];
    [WiseTracker initEnd];
    [WiseTracker urlRefererCheck:sourceApplication url:url];
```

### 4. 초기화
유니티 앱 실행시 최초 실행되는 MonoBehavior 상속받아 구현된 MainScene 클래스의 Awake() 함수에 다음과 같은 초기화 코드를 삽입해 주세요.

```c#
	#if UNITY_ANDROID && !UNITY_EDITOR
		WiseTracker.init(); // initialize 코드 삽입
		WiseTracker.startPage("유니크한 페이지 정보 입력"); // 페이지 호출           
	#elif UNITY_IOS && !UNITY_EDITOR // for ios
		WiseTracker.initialization("제공받은 앱키");
	#endif
```

