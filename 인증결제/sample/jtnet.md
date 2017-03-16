#1. PC 연동  
`IMP.request_pay(param, callback)`의 callback 함수를 활용하여 결제완료 통지를 받을 수 있습니다.  

- JTNet을 "기본PG사"로 설정해 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. (관리자 페이지에 설정된 정보를 기본으로 사용합니다)  
- 복수개의 PG설정을 해두신 경우 JTNet 결제창을 호출하시려면 `pg`파라메터를 `jtnet`로 설정해주세요.  


```javascript
IMP.request_pay({
    pg : 'jtnet',
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
    			msg += '\n결제 금액 : ' + rsp.paid_amount;
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

#2. 모바일 브라우저 연동
## 2.1 결제창 호출하기(m\_redirect\_url 설정)

JTNet 모바일 브라우저 결제에서는 페이지 이동이 자동으로 이뤄지기 때문에 `IMP.request_pay(param)`호출을 하셔야하고, PC와 달리 `callback` 함수를 사용하실 수 없습니다.  
대신, `param`에 `m_redirect_url`파라메터를 설정해 결제완료를 인지하실 수 있습니다.  

```javascript
IMP.request_pay({
    pg : 'jtnet',
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
`m_redirect_url`을 `https://www.my-service.com/payments/complete/mobile`로 지정하면, 실제 결제완료 후 이동하는 URL은 `https://www.my-service.com/payments/complete/mobile?imp_uid={아임포트거래고유번호}&merchant_uid={merchant_uid}`와 같이 imp\_uid, merchant\_uid값이 Query String에 추가된 URL이 됩니다.  
때문에, **https://www.my-service.com/payments/complete/mobile** 주소에 대한 GET request를 처리하는 서버단 로직에서 다음과 같이 결제완료 정보를 조회하면 됩니다.  

```
imp_uid = extract_GET_value('imp_uid')

payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' AND payment_result.amount == amount_to_be_paid
	success_post_process(payment_result) //결제까지 성공적으로 완료
ELSE IF payment_result.status == 'ready' AND payment.pay_method == 'vbank'
	vbank_number_assigned(payment_result) //가상계좌 발급성공
ELSE
	fail_post_process(payment_result) //결제실패 처리
```

#3. 모바일 WebView 연동  
## 3.1 안드로이드  
앱 내 결제의 경우 WebView를 활용해 결제가 이뤄지기 때문에 모바일 브라우저와 거의 동일한 프로세스를 가지게 됩니다. 다만, 앱 간 이동을 위해 URL Scheme처리를 위한 Native Code가 추가로 필요하며, `IMP.request_pay(param, callback)`호출 시 `param.app_scheme`파라메터를 통해 AndroidManifest.xml에 선언된 나의 scheme값을 지정해야 합니다.  

```javascript
IMP.request_pay({
    pg : 'jtnet',
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
    app_scheme : 'iamporttest' //개발 중인 앱에 정의된 URL scheme을 지정합니다. ://는 포함하지 않습니다.
});
```

### 3.1.1 WebView에서 외부 앱 호출  
카드사별 다양한 앱 호출 및 설치 요구에 대응하기 위해 다음의 코드가 추가되어야 합니다.  
[WebViewClient](https://developer.android.com/reference/android/webkit/WebViewClient.html) 참조  

```java
@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {

    if (!url.startsWith("http://") && !url.startsWith("https://") && !url.startsWith("javascript:")) {
        Intent intent = null;

        try {
            intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME); //IntentURI처리
            Uri uri = Uri.parse(intent.getDataString());

            activity.startActivity(new Intent(Intent.ACTION_VIEW, uri)); //해당되는 Activity 실행
            return true;
        } catch (URISyntaxException ex) {
            return false;
        } catch (ActivityNotFoundException e) {
            if ( intent == null )   return false;

            if ( handleNotFoundPaymentScheme(intent.getScheme()) )  return true; //설치되지 않은 앱에 대해 사전 처리(Google Play이동 등 필요한 처리)

            String packageName = intent.getPackage();
            if (packageName != null) { //packageName이 있는 경우에는 Google Play에서 검색을 기본
                activity.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + packageName)));
                return true;
            }

            return false;
        }
    }

    return false;
}
```
### 3.1.2 외부 앱에서 결제인증 후 WebView로 복귀  

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


## 3.2 iOS  
앱 내 결제의 경우 WebView를 활용해 결제가 이뤄지기 때문에 모바일 브라우저와 거의 동일한 프로세스를 가지게 됩니다. 다만, 앱 간 이동을 위해 URL Scheme처리를 위한 Native Code가 추가로 필요하며, `IMP.request_pay(param, callback)`호출 시 `param.app_scheme`파라메터를 통해 `info.plist` 에 선언된 나의 `CFBundleURLSchemes` 값을 지정해야 합니다.(javascript는 안드로이드와 동일)  

```xml
<key>CFBundleURLSchemes</key>
<array>
	<string>iamporttest</string>
</array>
```
### 3.2.1 WebView에서 외부 앱 호출  
```objective-c
-(BOOL)webView:(UIWebView *)webView shouldStartLoadWithRequest:(NSURLRequest *)request navigationType:(UIWebViewNavigationType)navigationType
{
    //ISP 호출하는 경우
    if([URLString hasPrefix:@"ispmobile://"]) {
        NSURL *appURL = [NSURL URLWithString:URLString];
        if([[UIApplication sharedApplication] canOpenURL:appURL]) {
            [[UIApplication sharedApplication] openURL:appURL];
        } else {
            [self showAlertViewWithEvent:@"모바일 ISP가 설치되어 있지 않아\nApp Store로 이동합니다." tagNum:99];
            return NO;
        }
    }
}
```

canOpenURL() 함수 사용을 위해 info.plist에 다음과 같이 whitelist를 모두 추가해두시는 것을 권장합니다.  

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
</array>
```

