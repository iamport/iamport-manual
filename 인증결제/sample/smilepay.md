# 1. PG 설정  

## 1.1 테스트모드 설정  
![스마일페이-아임포트 시스템 설정](screenshot/smilepay.png)

- PG사 : [간편결제] 스마일페이 선택 후, 테스트모드 설정 ON상태로 두고 저장합니다.

## 1.2 상용모드 설정  

1. [아임포트 PG가입 페이지](http://www.iamport.kr)에서 스마일페이 가입 신청을 합니다. 
2. PG사 : [간편결제] 스마일페이 선택 후, 테스트모드 설정 OFF상태로 둡니다. 
3. 계약 후 LGCNS에서 발급받은 PG상점아이디(MID) / 서명키 / merchantEncKey / merchantHashKey 를 입력 후 저장합니다. 


# 2. PC, 모바일 브라우저 연동  

스마일페이의 경우 레이어를 통해 결제창이 제공되며 페이지 리디렉션이 없어 PC, 모바일 브라우저 환경에서 동일한 소스코드를 적용할 수 있습니다.  

페이지 리디렉션이 없기 때문에 PC, 모바일 모두 `IMP.request_pay(param, callback)`의 callback 함수를 활용할 수 있습니다.  

- 스마일페이를 "기본PG사"로 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. 
- 스마일페이 내 에서 결제수단이 선택되므로 `pay_method`파라메터는 무시됩니다. (생략 가능)


```javascript
IMP.request_pay({
    pg : 'smilepay',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
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

# 3. 모바일 WebView 연동  

웹페이지단 소스코드는 모바일 브라우저 결제연동과 동일하되 iOS결제의 경우 `app_scheme` 파라메터가 추가됩니다. (안드로이드결제시에는 해당 파라메터가 전달되어도 무시됨)

```javascript
IMP.request_pay({
    pg : 'smilepay',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    app_scheme : 'your-app-scheme' //가맹점의 앱 스키마 값
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

## 3.1 iOS  

웹페이지단 소스코드는 위와 같이 진행하면 되며, Native 코드와 관련해서 추가되어야 하는 내용이 있습니다. 

### 스마일페이 앱 스키마 white-list 추가  
인증단계에서 스마일페이 앱으로 이동을 위해 info.plist 파일에 아래의 내용이 추가되어야 합니다.  

```
<key>LSApplicationQueriesSchemes</key>
<array>
	<string>smilepayapp</string>
</array>
```

### 가맹점 앱스키마 사전 등록 필요  
스마일페이 앱으로 이동 후 가맹점의 앱으로 복귀할 때, LGCNS서버에 사전 등록된 앱스카마만 호출됩니다.  
`IMP.request_pay()` 에 `app_scheme` 파라메터로 전달한 값을 LGCNS계약 담당자를 통해 사전 등록해주시기 바랍니다.  

### 3rd party 쿠키 허용  
원활한 스마일페이 인증단계 데이터 통신을 위해 결제가 진행되는 WebView에 쿠키허용 설정이 되어야 합니다.  

```
NSHTTPCookieStorage *cookieStorage = [NSHTTPCookieStorage sharedHTTPCookieStorage];
[cookieStorage setCookieAcceptPolicy:NSHTTPCookieAcceptPolicyAlways];
```


## 3.2 안드로이드   

iOS와 달리 스마일페이 앱 이동 후 가맹점 앱으로 복귀할 때 명시적으로 `app_scheme` 인텐트가 호출되지 않습니다. 스마일페이 앱 종료를 통해 기존에 결제를 진행하던 가맹점의 앱으로 Activity Stack 상 복귀하는 것 뿐이기 때문에  
`app_scheme` 파라메터 전달 및 AndroidManifest.xml 에 `intent-filter`를 정의할 필요가 없습니다.  

### WebViewClient 구현  
스마일페이 앱 호출을 위해 인텐트에 대응할 일반적인 수준의 `WebViewClient` 구현이 필요합니다.  

```
public class SmilepayViewClient extends WebViewClient {

	@Override
	public boolean shouldOverrideUrlLoading(WebView view, String url) {
		if (!url.startsWith("http://") && !url.startsWith("https://") && !url.startsWith("javascript:")) {
            Intent intent = null;

            try {
                intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME); //IntentURI처리
                Uri uri = Uri.parse(intent.getDataString());

                activity.startActivity(new Intent(Intent.ACTION_VIEW, uri));
                return true;
            } catch (URISyntaxException ex) {
                return false;
            } catch (ActivityNotFoundException e) {
                if (intent == null) return false;

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

}
```

### 3rd party 쿠키 허용  
원활한 스마일페이 인증단계 데이터 통신을 위해 결제가 진행되는 WebView에 쿠키허용 설정이 되어야 합니다.  

```
WebSettings settings = webView.getSettings();

if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
	settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
	CookieManager cookieManager = CookieManager.getInstance();
	cookieManager.setAcceptCookie(true);
	cookieManager.setAcceptThirdPartyCookies(mainWebView, true);
}
```