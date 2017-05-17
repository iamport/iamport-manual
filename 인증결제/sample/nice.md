# 1. PC 연동  
`IMP.request_pay(param, callback)`의 callback 함수를 활용하여 결제완료 통지를 받을 수 있습니다.  

- 나이스정보통신을 "기본PG사"로 설정해 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. (관리자 페이지에 설정된 정보를 기본으로 사용합니다)  
- 복수 PG연동 상황에서 `pg`파라메터를 설정하실 때는 `nice`를 사용하시면 됩니다.  


```javascript
IMP.request_pay({
    pg : 'nice',
    pay_method : 'card', //card(신용카드), trans(실시간계좌이체), vbank(가상계좌), phone(휴대폰소액결제)
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
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

# 2. 모바일 브라우저 연동  
## 2.1 결제창 호출하기(m\_redirect\_url 설정)

나이스정보통신 모바일 브라우저 결제에서는 페이지 이동이 자동으로 이뤄지기 때문에 `IMP.request_pay(param)`호출을 하셔야하고, PC와 달리 `callback` 함수를 사용하실 수 없습니다.  
대신, `param`에 `m_redirect_url`파라메터를 설정해 결제완료를 인지하실 수 있습니다.  

```javascript
IMP.request_pay({
    pg : 'nice', 
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : 'https://www.my-service.com/payments/complete/mobile'
});
```

## 2.2 서버단에서 결제완료 인지하기
**https://www.my-service.com/payments/complete/mobile** 주소에 대한 GET request를 처리하는 서버단 로직에서 다음과 같이 결제완료 정보를 조회하면 됩니다.  

```
imp_uid = extract_GET_value_from_url('imp_uid')

payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' && payment_result.amount == amount_to_be_paid
	success_post_process(payment_result)
ELSE
	fail_post_process(payment_result)
```

## 2.3 나이스 모바일 결제 신규버전 적용하기  
기본 적용되는 나이스 모바일 결제모듈은, 카드사 결제가 완료된 후 나이스정보통신으로부터 결제 완료 통지가 별도의 세션으로 전달되는 방식이므로 사용자의 환경에 따라 간혹 결제 정보가 제대로 도달하지 못하는 경우가 있었습니다. 
이를 보완하기 위해 나이스정보통신에서 신규 모바일 버전을 제공하고있으며 아임포트에도 이 모듈이 동시에 적용되어있습니다. 다만, legacy를 고려해 기존 모듈을 기본 적용하고, 특정 파라메터(`niceMobileV2`)를 통해 신규버전 모드로 결제를 진행하실 수 있도록 제공합니다. 

다음과 같이 `niceMobileV2 : true`를 설정하시면 내부적으로는 신규 모듈 버전으로 동작합니다.  
(물론, 아임포트 연동 방식에 있어서는 해당 파라메터를 제외하면 차이가 없습니다)

```javascript
IMP.request_pay({
    pg : 'nice', 
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    niceMobileV2 : true,
    m_redirect_url : 'https://www.my-service.com/payments/complete/mobile'
});
```



# 3. 모바일 WebView 연동  

앱간 연동을 위해 아래와 같이 `app_scheme`파라메터를 추가합니다. (그 외에는 모바일 브라우저 결제연동과 동일합니다)  

```javascript
IMP.request_pay({
    pg : 'nice',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : 'https://www.my-service.com/payments/complete/mobile',
    app_scheme : 'iamporttest' //개발 중인 앱에 정의된 URL scheme을 지정합니다
});
```


## 3.1 안드로이드  
- 샘플 프로젝트 : [https://github.com/iamport/iamport-nice-android](https://github.com/iamport/iamport-nice-android)  

앱 내 결제의 경우 WebView를 활용해 결제가 이뤄지기 때문에 모바일 브라우저와 동일한 프로세스를 가지게 됩니다. 다만, 다음과 같은 추가 처리가 필요합니다. 

### 3.1.1 URL Scheme  
#### AndroidManifest.xml에 intent-filter 정의  

```xml
<activity
	android:name=".MainActivity"
	android:label="@string/app_name">
	
	<intent-filter>
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />
		
		<data android:scheme="iamporttest" /> <!-- 예시로 iamporttest로 설정. My앱의 특징을 나타내는 고유의 scheme을 사용하세요 -->
	</intent-filter>
	
</activity>
```

#### WebViewClient 적용  
```java
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
	
	if (!url.startsWith("http://") && !url.startsWith("https://") && !url.startsWith("javascript:")) {
		Intent intent = null;
		
		try {
			/* START - BankPay(실시간계좌이체)에 대해서는 예외적으로 처리 */
			if ( url.startsWith(PaymentScheme.BANKPAY) ) {
				try {
					String reqParam = makeBankPayData(url);
					
					intent = new Intent(Intent.ACTION_MAIN);
                    intent.setComponent(new ComponentName("com.kftc.bankpay.android","com.kftc.bankpay.android.activity.MainActivity"));
                    intent.putExtra("requestInfo",reqParam);
                    activity.startActivityForResult(intent,RESCODE);
                    
                    return true;
				} catch (URISyntaxException e) {
					return false;
				}
			}
			/* END - BankPay(실시간계좌이체)에 대해서는 예외적으로 처리 */
			
			intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME); //IntentURI처리
			Uri uri = Uri.parse(intent.getDataString());
			
			activity.startActivity(new Intent(Intent.ACTION_VIEW, uri));
			return true;
		} catch (URISyntaxException ex) {
			return false;
		} catch (ActivityNotFoundException e) {
			if ( intent == null )	return false;
			
			if ( handleNotFoundPaymentScheme(intent.getScheme()) )	return true;
			
			String packageName = intent.getPackage();
	        if (packageName != null) {
	            activity.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + packageName)));
	            return true;
	        }
	        
	        return false;
		}
	}
	
	return false;
}
```

### 3.1.2 Cookie 설정  
나이스정보통신 모듈과 카드사 모듈 간 연동과정에서 쿠키가 사용되는 경우가 있습니다. 때문에, WebView에 대해 다음과 같은 설정을 해주셔야 원활한 결제가 이뤄지게 됩니다.  
commit : [3a3abff](https://github.com/iamport/iamport-nice-android/commit/3a3abff1f084c8d4da31c3f8edb36c278f45121c) 참조

```java
if (android.os.Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
	CookieManager cookieManager = CookieManager.getInstance();
    cookieManager.setAcceptCookie(true);
    cookieManager.setAcceptThirdPartyCookies(mainWebView, true);
}
```

## 3.2 iOS  
- 샘플 프로젝트(Swift) : [https://github.com/JosephNK/SwiftyIamport](https://github.com/JosephNK/SwiftyIamport)  
- 샘플 프로젝트(Objective-C) : [https://github.com/iamport/iamport-nice-ios](https://github.com/iamport/iamport-nice-ios)  

### 3.2.1 URL Scheme  
Xcode Build Info에 다음과 같이 `URL Scheme` 정의  
![Xcode Capture](screenshot/nice_xcode_scheme.png)

### 3.2.2 Info.plist whitelist  
앱 연동 과정에서 카드사의 앱카드, ISP앱 등으로 이동이 이뤄지는 경우가 있습니다. iOS10이후 보안정책으로, 이동이 일어날 수 있는 앱들의 URL scheme에 대한 whitelist를 미리 정의해두어야 합니다.  

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
    <string>kftc-bankpay</string> <!-- 계좌이체 -->
    <string>ispmobile</string> <!-- ISP모바일 -->
    <string>itms-apps</string> <!-- 앱스토어 -->
    <string>hdcardappcardansimclick</string> <!-- 현대카드-앱카드 -->
    <string>smhyundaiansimclick</string> <!-- 현대카드-공인인증서 -->
    <string>shinhan-sr-ansimclick</string> <!-- 신한카드-앱카드 -->
    <string>smshinhanansimclick</string> <!-- 신한카드-공인인증서 -->
    <string>kb-acp</string> <!-- 국민카드-앱카드 -->
    <string>mpocket.online.ansimclick</string> <!-- 삼성카드-앱카드 -->
    <string>ansimclickscard</string> <!-- 삼성카드-온라인결제 -->
    <string>ansimclickipcollect</string> <!-- 삼성카드-온라인결제 -->
    <string>vguardstart</string> <!-- 삼성카드-백신 -->
    <string>samsungpay</string> <!-- 삼성카드-삼성페이 -->
    <string>scardcertiapp</string> <!-- 삼성카드-공인인증서 -->
    <string>lottesmartpay</string> <!-- 롯데카드-모바일결제 -->
    <string>lotteappcard</string> <!-- 롯데카드-앱카드 -->
    <string>cloudpay</string> <!-- 하나카드-앱카드 -->
    <string>nhappvardansimclick</string> <!-- 농협카드-앱카드 -->
    <string>nonghyupcardansimclick</string> <!-- 농협카드-공인인증서 -->
    <string>citispay</string> <!-- 씨티카드-앱카드 -->
    <string>citicardappkr</string> <!-- 씨티카드-공인인증서 -->
    <string>kakaotalk</string> <!-- 카카오톡 -->
</array>
```


### 3.2.3 Cookie 설정  
나이스정보통신 모듈과 카드사 모듈 간 연동과정에서 쿠키가 사용되는 경우가 있습니다. 때문에, 다음과 같은 설정을 해주셔야 원활한 결제가 이뤄지게 됩니다.  

```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    //iOS6에서 세션끊어지는 상황 방지하기 위해 쿠키 설정.
    [[NSHTTPCookieStorage sharedHTTPCookieStorage] setCookieAcceptPolicy:NSHTTPCookieAcceptPolicyAlways];
    
    return YES;
}
```