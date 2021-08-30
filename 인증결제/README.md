# PG사별 일반결제 연동 가이드

아임포트 결제 연동 매뉴얼의 [준비하기](https://docs.iamport.kr/prepare) 가이드에 따라 아임포트 계정을 생성하고, 다음 PG사 별 일반결제 연동 가이드를 참고하여 각 PG사 테스트 계정으로 테스트 결제를 해 볼수 있습니다.<Br />

ℹ️ 국내 전자결제 서비스의 단계별 흐름은 [한국결제의 특징](background.md)을 참고하세요.

다음 PG사별 가이드는 각 PG사에 해당하는 내용만 포함합니다. PG 공통 설정 및 샘플코드 등 연동 방법은 아래 내용을 참고하세요.

- [알리페이](./sample/alipay.md)
- [블루월넛](./sample/bluewalnut.md)
- [차이](./sample/chai.md)
- [다날](./sample/danal.md)
- [엑심베이](./sample/eximbay.md)
- [나이스페이먼츠](./sample/inicis.md)
- [JTNet](./sample/jtnet.md)
- [카카오페이](./sample/kakao.md)
- [NHN KCP](./sample/kcp.md)
- [KICC](./sample/kicc.md)
- [나이스페이먼츠](./sample/nice.md)
- [페이코](./sample/payco.md)
- [페이팔](./sample/paypal.md)
- [세틀뱅크](./sample/settle.md)
- [스마일페이](./sample/smilepay.md)
- [토스페이먼츠](./sample/uplus.md)
- [네이버페이](/NAVERPAY/sample/naverpay-pg.md)
  
# PG 공통 연동 방법

- [1. PC/모바일 웹에서 PG 연동하기](#pc-mobile)
	- [1.1 Callback 방식](#callback)
	- [1.2 리디렉션 방식](#redirect)
- [2. 모바일 앱 WebView에서 PG 연동하기](#webview)
	- [2.1 가맹점 앱 -> 외부 앱](#my-to-3rd)
		- [안드로이드](#my-to-3rd-android)
		- [iOS](#my-to-3rd-ios)
	- [2.2 외부 앱 -> 가맹점 앱](#3rd-to-my)
		- [안드로이드](#3rd-to-my-android)
		- [iOS](#3rd-to-my-ios)
	- [2.3 쿠키 설정](#cookie)	
		- [안드로이드](#cookie-android)
		- [iOS](#cookie-ios)

<a id="pc-mobile"></a>

## 1. PC/모바일 웹에서 PG 연동하기

각 PG사별로 결제창을 호출하고 결제 프로세스가 완료되면 결제정보 수신 등 처리 로직이 다음과 같이 **callback 또는 리디렉션**으로 실행됩니다.


ℹ️ 자세한 내용은 [일반결제 연동하기](https://docs.iamport.kr/implementation/payment#start-payment)를 참고하세요.

ℹ️ 아임포트를 사용하는 개발자가 오픈소스로 제공하는 [언어별 REST API 클라이언트 모듈](https://github.com/iamport/iamport-rest-client)을 활용하면 보다 쉽게 아임포트 REST API를 호출할 수 있습니다.

<a id="callback"></a>

### 1.1 Callback 방식

- Callback 방식으로 결제창을 호출하는 샘플코드 `client-side`

```javascript
IMP.request_pay({
    pg : 'html5_inicis',
    pay_method : 'card',
    merchant_uid: "order_monthly_0001", // 상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
}, function(rsp) {
    if ( rsp.success ) {
    	//[1] 서버단에서 결제정보 조회를 위해 jQuery ajax로 imp_uid 전달하기
    	jQuery.ajax({
    		url: "/payments/complete", //cross-domain error가 발생하지 않도록 주의해주세요
    		type: 'POST',
    		dataType: 'json',
    		data: {
	    		imp_uid : rsp.imp_uid
	    		//기타 필요한 데이터가 있으면 추가 전달
    		}
    	}).done(function(data) {
    		//[2] 서버에서 REST API로 결제정보확인 및 서비스루틴이 정상적인 경우
    		if ( everythings_fine ) {
    			var msg = '결제가 완료되었습니다.';
    			msg += '\n고유ID : ' + rsp.imp_uid;
    			msg += '\n상점 거래ID : ' + rsp.merchant_uid;
    			msg += '\결제 금액 : ' + rsp.paid_amount;
    			msg += '카드 승인번호 : ' + rsp.apply_num;
    			
    			alert(msg);
    		} else {
    			//[3] 아직 제대로 결제가 되지 않았습니다.
    			//[4] 결제된 금액이 요청한 금액과 달라 결제를 자동취소처리하였습니다.
    		}
    	});
    } else {
        var msg = '결제에 실패하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;
        
        alert(msg);
    }
});
```

- Ajax POST 방식으로 전달된 결제정보를 추출하여 후처리 하는 의사코드 `server-side`

```
imp_uid = extract_POST_value_from_url('imp_uid') //post ajax request로부터 imp_uid확인

payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' AND payment_result.amount == amount_to_be_paid
	success_post_process(payment_result) //결제까지 성공적으로 완료
ELSE IF payment_result.status == 'ready' AND payment.pay_method == 'vbank'
	vbank_number_assigned(payment_result) //가상계좌 발급성공
ELSE
	fail_post_process(payment_result) //결제실패 처리

```


<a id="redirect"></a>

### 1.2 리디렉션 방식

- 리디렉션 방식으로 결제창을 호출하는 샘플코드 `client-side`

```javascript
IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans',
    merchant_uid: "order_monthly_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
}, function(rsp) {
    if ( !rsp.success ) {
    	//결제 시작 페이지로 리디렉션되기 전에 오류가 난 경우
        var msg = '오류로 인하여 결제가 시작되지 못하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

- Redirect URL의 쿼리 스트링으로 전달 된 결제정보를 추출하여 후처리 하는 의사코드 `server-side`

```
imp_uid = extract_GET_value_from_url('imp_uid') //GET query string으로부터 imp_uid확인
//merchant_uid = extract_GET_value_from_url('merchant_uid') //또는, GET query string으로부터 merchant_uid확인

payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' AND payment_result.amount == amount_to_be_paid
	success_post_process(payment_result) //결제까지 성공적으로 완료
ELSE IF payment_result.status == 'ready' AND payment.pay_method == 'vbank'
	vbank_number_assigned(payment_result) //가상계좌 발급성공
ELSE
	fail_post_process(payment_result) //결제실패 처리
```

<a id="webview"></a>

## 2. 모바일 앱 WebView에서 PG 연동하기

아임포트에서 제공하는 **모바일 네이티브 SDK**를 사용하면 Android/iOS 네이티브 앱에서 편리하게 결제 연동을 할 수 있습니다.

- 아임포트 Android SDK : https://github.com/iamport/iamport-android
- 아임포트 iOS SDK : https://github.com/iamport/iamport-ios

<a id="app_scheme"></a>
PC/모바일 웹 연동의 [리디렉션 방식](#redirect)과 동일하게 앱내 WebView에서 각 PG사의 결제창을 호출하고 결제 승인 후처리를 합니다. 단, 결제 승인 후 가맹점 앱으로 복귀하기 위해 `IMP.request_pay(param, callback)` 함수의 `param.app_scheme`파라미터에 다음과 같이 **가맹점 앱의 scheme**값을 지정해야 합니다.

```javascript
IMP.request_pay({
	/*...중략... */
	app_scheme: 'iamporttest' // 가맹점 앱의 URL scheme, '://'는 포함하지 않습니다.
})
```

결제 수단별 인증은 다음과 같이 앱간 이동을 위한 설정 및 로직을 구성해야 합니다. 

1. 가맹점 앱 -> 외부 앱으로 이동
2. 외부 앱 -> 가맹점 앱으로 이동


<a id="my-to-3rd"></a>

## 2.1 가맹점 앱 -> 외부 앱

### 대표 외부 앱의 URL Scheme

| URL Scheme | App |
| ---------- | - |
| ansimclick | 삼성카드-온라인결제 |
| ansimclickipcollect | 삼성카드-온라인결제 |
| ansimclickscard | 삼성카드-온라인결제 |
| cardusim |  |
| citicardappkr | 씨티카드-공인인증 앱 |
| citispay | 씨티카드-앱카드 |
| cloudpay | 하나카드-앱카드 |
| droidxantivirus | |
| hdcardappcardansimclick | 현대카드-앱카드 |
| ispmobile | ISP모바일 |
| itms-apps | 앱스토어 |
| kakaotalk | 카카오페이 |
| kb-acp | 국민카드-앱카드 |
| kftc-bankpay | 계좌이체 |
| lotteappcard | 롯데카드-앱카드 |
| lottesmartpay | 롯데카드-모바일결제 |
| Lpayapp | L.pay 앱 |
| mpocket.online.ansimclick | 삼성카드-앱카드 |
| nhallonepayansimclick | 농협-올원페이 |
| nhappcardansimclick | 농협-앱카드 |
| nonghyupcardansimclick | 농협카드-공인인증 앱 |
| payco | 페이코 |
| samsungpay | 삼성카드-삼성페이 |
| scardcertiapp | 삼성카드-공인인증서 |
| shinhan-sr-ansimclick | 신한카드-앱카드 |
| smhyundaiansimclick | 현대카드-공인인증 앱 |
| smshinhanansimclick | 신한카드-공인인증 앱 |
| vguard | |
| vguardstart | 삼성카드-백신 |
| Wooripay | 우리카드-앱카드 |

### 2.1.a 안드로이드 <a id="my-to-3rd-android"></a>

가맹점 앱의 WebView에서 PG사별 앱 호출 및 미설치 체크 로직을 구현합니다.<Br />

[WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient.html) 클래스의 `shouldOverrideUrlLoading` 메소드를 다음과 같이 재정의하여 구현합니다.


```java
public class MyViewClient extends WebViewClient {

	@Override
	public boolean shouldOverrideUrlLoading(WebView view, String url) {
		if (!url.startsWith("http://") && !url.startsWith("https://") && !url.startsWith("javascript:")) {
			//외부 앱에 대한 URL scheme 대응
			Intent intent = null;

			try {
				intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME); //IntentURI처리
				Uri uri = Uri.parse(intent.getDataString());

				activity.startActivity(new Intent(Intent.ACTION_VIEW, uri));
				return true;
			} catch (URISyntaxException ex) {
				return false;
			} catch (ActivityNotFoundException e) {
				if ( intent == null )	return false;

				//설치되지 않은 앱에 대해 market이동 처리
				if ( handleNotFoundPaymentScheme(intent.getScheme()) )	return true;

				//handleNotFoundPaymentScheme()에서 처리되지 않은 것 중, url로부터 package정보를 추출할 수 있는 경우 market이동 처리
				String packageName = intent.getPackage();
				if (packageName != null) {
					activity.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + packageName)));
					return true;
				}

				return false;
			}
		}
	}
}
```

### 2.1.b iOS <a id="my-to-3rd-ios"></a>

외부 앱으로 이동 시 별도 처리과정이 필요 없으며 해당 앱을 **white-list에 등록**하면 확인 팝업창을 통해 외부 앱으로 이동합니다.  
<Br />
iOS 보안 정책상 외부 호출될 URL scheme을 `info.plist` 파일의 `LSApplicationQueriesSchemes`에 추가해야 외부 앱 실행 확인창이 열립니다.

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>kftc-bankpay</string>
	...
</array>
```

<a id="3rd-to-my"></a>

## 2.2 외부 앱 -> 가맹점 앱

외부 앱에서 결제가 완료되면 가맹점 앱에서 정의한 URL scheme을 호출하여 가맹점 앱으로 다시 돌아올 수 있습니다.

### 2.2.a 안드로이드 <a id="3rd-to-my-android"></a>

ℹ️ 안드로이드는 Activity가 종료되면 Activity Stack(Task)에서 이전 Activity로 자동으로 이동하는 특성이 있습니다. 이러한 특성을 활용하여 KG이니시스의 경우, URL scheme을 사용하지 않아도 동작에 문제가 없도록 설계되었습니다.

#### 가맹점 앱의 URL scheme 정의하기

`AndroidManifest.xml` 파일의 `intent-filter`에 [app_scheme](#app_scheme)에 지정한 가맹점 앱 URL scheme을 다음과 같이 추가합니다.

```xml
<activity
	android:name=".MainActivity"
	android:label="@string/app_name">

	<intent-filter>
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />

		<data android:scheme="iamporttest" /> <!-- 예시로 iamporttest로 설정. 가맹점 앱의 특징을 나타내는 고유의 scheme을 사용하세요 -->
	</intent-filter>

</activity>
```

#### URL scheme에 의해 실행된 Activity에서 Intent 처리하기

결제 인증 후 외부 앱이 URL scheme에 해당하는 Intent를 호출하면 가맹점 앱의 지정된 Activity가 실행됩니다. Activity의 `OnCreate` 메소드를 재정의하고 **URL scheme과 함께 전달된 가맹점 앱 URL**을 호출하여 가맹점 앱으로 복귀합니다.

URL scheme 예시: `iamporttest://://https://web.nicepay.co.kr/smart/card/isp/ispResult.jsp?tid=nictest00m01011604160159435894`


```java
private final String APP_SCHEME = "iamporttest://"; //AndroidManifest.xml에서 정의한 것과 동일한 URL scheme사용(redirect URL 추출을 위한 용도)

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	/*...중략...*/

	Intent intent = getIntent();
	Uri intentData = intent.getData();

	if ( intentData != null ) {
		//isp 인증 후 복귀했을 때 결제 후속조치
		String url = intentData.toString();
		if ( url.startsWith(APP_SCHEME) ) {
			//가맹점 앱의 WebView가 표시해야 할 웹 컨텐츠의 주소가 전달됩니다.
			String redirectURL = url.substring(APP_SCHEME.length()+3);
			mainWebView.loadUrl(redirectURL);
		}
	}
}
```

### 2.2.b iOS <a id="3rd-to-my-ios"></a>

#### 가맹점 앱의 URL scheme 정의하기


XCode의 Build Info 탭에 [app_scheme](#app_scheme)에 지정한 가맹점 앱 URL scheme을 다음과 같이 추가합니다.  

![Xcode Capture](sample/screenshot/nice_xcode_scheme.png)

```xml
<key>CFBundleURLSchemes</key>
<array>
	<string>iamporttest</string>
</array>
```

## 3. 쿠키 설정하기 <a id="cookie"></a>

PG사 모듈과 카드사 모듈 간 연동과정에서 쿠키가 사용될 수 있으므로 원할한 결제를 위해 WebView에 다음과 같이 설정합니다.

## 3.1 Android

관려 Git commit : [3a3abff](https://github.com/iamport/iamport-nice-android/commit/3a3abff1f084c8d4da31c3f8edb36c278f45121c)

```java
WebSettings settings = webView.getSettings();

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
	settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
	CookieManager cookieManager = CookieManager.getInstance();
	cookieManager.setAcceptCookie(true);
	cookieManager.setAcceptThirdPartyCookies(mainWebView, true);
}
```


### 3.2 iOS  

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    //iOS6에서 세션끊김 방지를 위한 쿠키 설정.
    [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookieAcceptPolicy:NSHTTPCookieAcceptPolicyAlways];
    
    return YES;
}

```
