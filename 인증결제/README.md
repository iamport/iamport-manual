# PG사별 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/README.md)  

아임포트 결제 연동 매뉴얼의 [준비하기](https://docs.iamport.kr/prepare) 가이드에 따라 아임포트 계정을 생성하고, 다음 PG사 별 일반결제 연동 가이드를 참고하여 각 PG사 테스트 계정으로 테스트 결제를 해 볼수 있습니다.<Br />

ℹ️ 국내 전자결제 서비스의 단계별 흐름은 [한국결제의 특징](background.md)을 참고하세요.

다음 PG사별 가이드는 각 PG사에 해당하는 내용만 포함합니다. PG 공통 설정 및 샘플코드 등 연동 방법은 아래 내용을 참고하세요.

- [알리페이](./sample/alipay.md)
- [블루월넛](./sample/bluewalnut.md)
- [차이](./sample/chai.md)
- [다날](./sample/danal.md)
- [엑심베이](./sample/eximbay.md)
- [KG이니시스](./sample/inicis.md)
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
- [PaymentWall](./sample/paymentwall.md)
- [토스간편결제](./sample/tosspay.md)
  
# PG 공통 연동 방법

- [1. PC/모바일 웹에서 PG 연동하기](#pc-mobile)
	- [1.1 Callback 방식](#callback)
	- [1.2 리디렉션 방식](#redirect)
- [2. 모바일 앱 WebView에서 PG 연동하기](#webview)
	- [2.1 가맹점 앱 -> 외부 앱](#my-to-3rd)
		- [2.1.a 안드로이드](#my-to-3rd-android)
		- [2.1.b iOS](#my-to-3rd-ios)
	- [2.2 외부 앱 -> 가맹점 앱](#3rd-to-my)
		- [2.2.a 안드로이드](#3rd-to-my-android)
		- [2.2.b iOS](#3rd-to-my-ios)
	- [2.3 쿠키 설정](#cookie)	
		- [2.3.a 안드로이드](#cookie-android)
		- [2.3.b iOS](#cookie-ios)
	- [2.4 Alert/Confirm 창 띄우기](#alert)	
		- [2.4.a 안드로이드](#alert-android)
		- [2.4.b iOS](#alert-ios)


<a id="pc-mobile"></a>

## 1. PC/모바일 웹에서 PG 연동하기

각 PG사별로 결제창을 호출하고 결제 프로세스가 완료되면 결제정보 수신 등 처리 로직이 다음과 같이 **callback 또는 리디렉션**으로 실행됩니다. *결제창이 팝업창으로 열리는 경우, 팝업 차단 또는 사용자 확인창을 회피하기 위해 [IMP.request_pay(param, callbak)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay) 호출은 `onClick`과 같이 사용자 액션에 대한 핸들러에서 처리되어야 합니다.*  

ℹ️ 자세한 내용은 [일반결제 연동하기](https://docs.iamport.kr/implementation/payment#start-payment)를 참고하세요.

ℹ️ 아임포트를 사용하는 개발자가 오픈소스로 제공하는 [언어별 REST API 클라이언트 모듈](https://github.com/iamport/iamport-rest-client)을 활용하면 보다 쉽게 아임포트 REST API를 호출할 수 있습니다.

<a id="callback"></a>

### 1.1 Callback 방식

- Callback 방식으로 결제창을 호출하는 샘플코드 `client-side`

```javascript
IMP.request_pay({
    pg : 'html5_inicis',
    pay_method : 'card',
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호
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
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete
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

ℹ️ 아임포트에서 제공하는 [모바일 네이티브 SDK](https://docs.iamport.kr/sdk/mobile-sdk)를 사용하면 Android/iOS 네이티브 앱에서 편리하게 결제 연동을 할 수 있습니다.  

PC/모바일 웹 연동의 [리디렉션 방식](#redirect)과 동일하게 앱내 WebView에서 각 PG사의 결제창을 호출하고 결제 승인 후처리를 합니다.  

<a id="app_scheme"></a>
**iOS의 경우**, 결제 승인 후 가맹점 앱으로 복귀하기 위해 [IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay) 함수의 `param.app_scheme`파라미터에 **가맹점 앱의 scheme**값을 지정해야 합니다.  

`app_scheme`은 다음의 두가지 형식을 지원합니다.

-	`iamporttest` : URL scheme 값만 지정. 아임포트에서는 `iamporttest://`로 호출됩니다.
-	`iamporttest://path?query` : 앱을 통해 전달받을 파라메터를 함께 지정. 지정한 값으로 호출됩니다.


```javascript
IMP.request_pay({
	/*...중략... */
	app_scheme: 'iamporttest' // 가맹점 앱의 URL scheme
})
```

결제 수단별 인증은 다음과 같이 앱간 이동을 위한 설정 및 로직을 구성해야 합니다.  

1. 가맹점 앱 -> 외부 앱으로 이동
2. 외부 앱 -> 가맹점 앱으로 이동


<a id="my-to-3rd"></a>

## 2.1 가맹점 앱 -> 외부 앱

### 2.1.a 안드로이드 <a id="my-to-3rd-android"></a>

ℹ️ 가맹점 앱의 WebView에서 PG사별 앱 호출 및 미설치 체크 로직을 구현합니다.<Br />


ℹ️ [Android 11 보안정책에 따른 앱 패키지 등록](https://guide.iamport.kr/df95fe31-c0d0-4215-8c4f-1d0b539fad88) 이 필요할 수 있으니 먼저 링크의 가이드를 참조하시기 바랍니다.


[WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient.html) 클래스의 `shouldOverrideUrlLoading` 메소드를 다음과 같이 재정의하여 구현합니다.

- java
```java
// java 예시
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


- kotlin
```kotlin
// kotlin 예시
// 예시코드이며 가맹점 구현에 따라 다를 수 있습니다.
override fun shouldOverrideUrlLoading(view: WebView?, request: WebResourceRequest?): Boolean {
    request?.url?.let {
        if (it.scheme == "about") {
            return true // 이동하지 않음
        }
        val urlStr = it.toString()
        if (!URLUtil.isNetworkUrl(urlStr) && !URLUtil.isJavaScriptUrl(urlStr)) {
            openPaymentApp(urlStr) // 앱이동
            return true
        }
    }
    return super.shouldOverrideUrlLoading(view, request)
}

fun openPaymentApp(url: String) {
    Intent.parseUri(url, Intent.URI_INTENT_SCHEME)?.let { intent: Intent ->
        runCatching {
            startActivity(intent) // 앱 이동
        }.recoverCatching {
            // 앱이동에 실패(미설치)시 앱스토어로 이동
            val packageName = intent.getPackage()
            if (!packageName.isNullOrBlank()) {
                startActivity(Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=$packageName")))
            }
        }
    }
}
```


### 2.1.b iOS <a id="my-to-3rd-ios"></a>

iOS 보안 정책상 외부 호출될 URL scheme을 `info.plist` 파일의 `LSApplicationQueriesSchemes`에 추가해야 외부 앱으로 이동할 수 있습니다.

<details>
<summary>대표 외부 앱의 URL Scheme 펼쳐보기</summary>

| URL Scheme | App |
| ---------- | - |
| ansimclick | 삼성카드-온라인결제 |
| ansimclickipcollect | 삼성카드-온라인결제 |
| ansimclickscard | 삼성카드-온라인결제 |
| chaipayment | 차이앱 |
| citicardappkr | 씨티카드-공인인증 앱 |
| citimobileapp | 씨티카드-간편결제 |
| citispay | 씨티카드-앱카드 |
| cloudpay | 하나카드-앱카드 |
| com.wooricard.wcard | 우리WON페이 |
| hanamopmoasign | 하나카드-공인인증 앱 |
| hanawalletmembers | 하나카드-하나멤버스 월렛 |
| hdcardappcardansimclick | 현대카드-앱카드 |
| hyundaicardappcardid | 현대카드 |
| ispmobile | ISP모바일 |
| itms-apps | 앱스토어 |
| kakaotalk | 카카오페이 |
| kb-acp | 국민카드-앱카드 |
| kb-auth | 국민 |
| kftc-bankpay | 뱅크페이 계좌이체 |
| lguthepay-xpay | 페이나우 |
| liivbank | 국민 Liiv M(리브모바일) |
| lmslpay | 롯데 L.pay 앱 |
| lotteappcard | 롯데카드-앱카드 |
| lottesmartpay | 롯데카드-모바일결제 |
| lpayapp | (구)롯데 L.pay 앱 |
| mpocket.online.ansimclick | 삼성카드-앱카드 |
| newsmartpib | 우리WON뱅킹 |
| nhallonepayansimclick | 농협-올원페이 |
| nhappcardansimclick | 농협-앱카드 |
| nonghyupcardansimclick | 농협카드-공인인증 앱 |
| payco | 페이코 |
| samsungpay | 삼성카드-삼성페이 |
| scardcertiapp | 삼성카드-공인인증서 |
| shinhan-sr-ansimclick | 신한카드-앱카드 |
| smhyundaiansimclick | 현대카드-공인인증 앱 |
| smshinhanansimclick | 신한카드-공인인증 앱 |
| supertoss | 토스 |
| vguardstart | 삼성카드-백신 |
| wooripay | 우리카드-앱카드 |
</details>

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>kftc-bankpay</string>
	...
</array>
```

또한, url 변경 과정에서 앱 url 을 감지했을 때, 웹뷰내 페이지 전환을 하지 않고 앱스킴을 실행시키려면 아래 코드를 구현하셔야 합니다.
```swift
// swift
func webView(
    _ webView: WKWebView,
    decidePolicyFor navigationAction: WKNavigationAction,
    decisionHandler: @escaping (WKNavigationActionPolicy) -> Void
) {
    if let url = navigationAction.request.url,
    url.scheme != "http" && url.scheme != "https" {
        UIApplication.shared.openURL(url)
        decisionHandler(.cancel)
    } else {
        decisionHandler(.allow)
    }
}
```

<a id="3rd-to-my"></a>

## 2.2 외부 앱 -> 가맹점 앱

외부 앱에서 결제가 완료되면 가맹점 앱에서 정의한 URL scheme을 호출하여 가맹점 앱으로 다시 돌아올 수 있습니다.

### 2.2.a 안드로이드 <a id="3rd-to-my-android"></a>

안드로이드는 Activity가 종료되면 Activity Stack(Task)에서 이전 Activity로 자동으로 이동하는 특성이 있어서 가맹점 앱의 URL scheme 정의 및 별도 처리가 필요하지 않습니다.

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

## 2.3 쿠키 설정하기 <a id="cookie"></a>

PG사 모듈과 카드사 모듈 간 연동과정에서 쿠키가 사용될 수 있으므로 원할한 결제를 위해 WebView에 다음과 같이 설정합니다.

### 2.3.a 안드로이드 <a id="cookie-android"></a>

 [참고 샘플코드](https://github.com/iamport/iamport-android/blob/ef0dfc7c096fa614fc6d2a94dfcea6c0e441b3bd/sdk/src/main/java/com/iamport/sdk/presentation/activity/BaseMain.kt#L33)

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
	mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW
	val cookieManager = CookieManager.getInstance()
	cookieManager.setAcceptCookie(true)
	cookieManager.setAcceptThirdPartyCookies(webView, true)
}
```


### 2.3.b iOS  <a id="cookie-ios"></a>

[참고 샘플코드](https://github.com/iamport/iamport-ios/blob/8b2780b286e2f94595e51584e9b3fe25a4f7a630/iamport-ios/Classes/Domain/Iamport.swift#L22)

```java
HTTPCookieStorage.shared.cookieAcceptPolicy = HTTPCookie.AcceptPolicy.always
```

## 2.4 Alert/Confirm 창 띄우기 <a id="alert"></a>

WebView의 웹페이지에서 발생하는 alert/confirm 창을 Android 또는 iOS 팝업으로 표시하는 로직을 구현해야 각 창이 열립니다.

### 2.4.a 안드로이드 <a id="alert-android"></a>

[WebChromeClient](https://developer.android.com/reference/android/webkit/WebChromeClient) 클래스의 다음 메소드를 재정의하여 alert과 confirm 창을 표시합니다.

- Alert : `onJsAlert`
- Confirm : `onJsConfirm`

[참고 샘플코드](https://github.com/iamport/iamport-android/blob/main/sdk/src/main/java/com/iamport/sdk/domain/IamportWebChromeClient.kt)

### 2.4.b iOS <a id="alert-ios"></a>

웹페이지에서 alert과 confirm 창 발생 시 호출되는 [WKUIDelegate](https://developer.apple.com/documentation/webkit/wkuidelegate) 프로토콜의 다음 함수에 해당 로직을 구현합니다.

- Alert : `webView(_ webView: WKWebView, runJavaScriptAlertPanelWithMessage:, initiatedByFrame:, completionHandler:)`
- Confirm : `webView(_ webView: WKWebView, runJavaScriptConfirmPanelWithMessage:, initiatedByFrame:, completionHandler:)`

[Alert 참고 샘플코드](https://github.com/iamport/iamport-ios/blob/main/iamport-ios/Classes/Presentation/WebViewController.swift#L479)

[Confirm 참고 샘플코드](https://github.com/iamport/iamport-ios/blob/8b2780b286e2f94595e51584e9b3fe25a4f7a630/iamport-ios/Classes/Presentation/WebViewController.swift#L491)
