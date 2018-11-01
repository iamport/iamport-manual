# 들어가기에 앞서  
KCP테스트모드에서 결제를 테스트하실 때에는 `KB카드` 결제승인이 이뤄지지 않습니다.  
테스트모드에 한하여 발생되는 문제이며, 다른 카드사로 시도하거나 실제 상점아이디로 진행할 때에는 발생되지 않는 문제입니다.  

# 1. PC 연동  

`IMP.request_pay(param, callback)`의 callback 함수를 활용하여 결제완료 통지를 받을 수 있습니다.  
`param`에서 지원되는 속성에 대한 상세한 내용은 [공통매뉴얼-2.1.1](https://github.com/iamport/iamport-manual/tree/master/%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C#211-param-속성공통-속성) 에서 확인바랍니다.  

- NHN KCP 를 "기본PG사"로 설정해 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. (관리자 페이지에 설정된 정보를 기본으로 사용합니다)  
- 복수개의 PG설정을 해두신 경우 NHN KCP 결제창을 호출하시려면 `pg`파라메터를 `kcp`로 설정해주세요.  

```javascript
IMP.init('imp33886024'); //아임포트 회원가입 후 발급된 가맹점 고유 코드를 사용해주세요. 예시는 KCP공식 아임포트 데모 계정입니다.

IMP.request_pay({
    pg : 'kcp', //웹표준 결제창 지원
    pay_method : 'card', //card(신용카드), samsung(삼성페이), trans(실시간계좌이체), vbank(가상계좌), phone(휴대폰소액결제)
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
    	//[1] 고객 입장에서 결제가 완료되었고, 아임포트 서버에도 결제완료로 상태가 변경된 상황
    	//가맹점 서버단에서 아임포트 REST API로 결제정보 조회를 위해 imp_uid를 운영하는 서버로 전달하기
    	jQuery.ajax({
    		url: "/payments/complete", //cross-domain error가 발생하지 않도록 주의해주세요
    		type: 'POST',
    		dataType: 'json',
    		data: {
	    		imp_uid : rsp.imp_uid
	    		//기타 필요한 데이터가 있으면 추가 전달
    		}
    	}).done(function(data) {
    		//[2] 서버에서 REST API로 결제정보확인 후 가맹점 DB에 결제완료로 상태 변경 등 서비스루틴이 정상적인 경우
    		if ( 결제확인 및 후처리 성공 ) {
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
    	//결제 실패 : error_msg속성에 실패 사유가 전달됨
        var msg = '결제에 실패하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;
        
        alert(msg);
    }
});
```


# 2. 모바일 브라우저 연동  
## 2.1 결제창 호출하기(m\_redirect\_url 설정)

NHN KCP 모바일 브라우저 결제에서는 결제시작과 동시에 페이지 이동이 자동으로 이뤄집니다. 
`IMP.request_pay(param, callback)`호출을 하더라도 PC와 달리 결제가 완료된 시점(성공/실패)에 `callback` 함수가 실행되지 않습니다.  
(단, 결제프로세스가 시작되기 전(페이지 이동 전) 검증 단계에서 문제가 있는 경우에는 호출이 됩니다)  

결제완료 시점에 `callback`이 호출될 수 없으므로 `param`에 `m_redirect_url`파라메터를 설정하여 결제가 완료되었을 때 이동할 URL을 지정할 수 있습니다.  
(PC결제와 유일한 차이점은 `m_redirect_url`파라메터의 존재이며, PC에서는 해당 파라메터를 무시하므로 코드 일관성을 위해 PC결제에도 `m_redirect_url`파라메터 전달이 무방합니다)  

`m_redirect_url`에 대한 보다 자세한 내용은 [공통매뉴얼-2.1.3](https://github.com/iamport/iamport-manual/tree/master/%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C#213-m_redirect_url)을 참고바랍니다.  

```javascript
IMP.init('imp33886024'); //아임포트 회원가입 후 발급된 가맹점 고유 코드를 사용해주세요. 예시는 KCP공식 아임포트 데모 계정입니다.

IMP.request_pay({
    pg : 'kcp',
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

결제가 완료되면(성공/실패) `m_redirect_url` + "?imp\_uid={imp\_uid}&merchant\_uid={merchant\_uid}&error\_msg={error\_msg}" 와 같이 Query String으로 추가 데이터가 URL로 함께 전달됩니다.  
`m_redirect_url` 자체가 Query String을 포함하고 있어도 보존된 상태로 Query String이 추가됩니다.  


# 3. 모바일 WebView 연동  
## 3.1 안드로이드  

앱 내 결제의 경우 WebView를 활용해 결제가 이뤄지기 때문에 모바일 브라우저와 거의 동일한 프로세스를 가지게 됩니다. 다만, 앱 간 이동을 위해 URL Scheme처리를 위한 Native Code가 추가로 필요하며, `IMP.request_pay(param, callback)`호출 시 `param.app_scheme`파라메터를 통해 AndroidManifest.xml에 선언된 나의 scheme값을 지정해야 합니다.  

```javascript
IMP.init('imp33886024'); //아임포트 회원가입 후 발급된 가맹점 고유 코드를 사용해주세요. 예시는 KCP공식 아임포트 데모 계정입니다.

IMP.request_pay({
    pg : 'html5_inicis',
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

### 3.1.1 WebView에서 외부 앱 호출을 위한 Native Code  

아래의 github repository에서 KCP 안드로이드 연동을 위한 샘플 프로젝트 확인이 가능합니다. 

[KCP 안드로이드 연동 샘플](https://github.com/iamport/kcp-android-graddle)


### 3.1.2 app\_scheme에 해당하는 intent-filter 정의  

`IMP.request_pay(param, callback)`호출 시 `param.app_scheme`에 지정한 것과 동일한 값으로 intent-filter를 정의해야 합니다.  

```xml
<activity
	android:name=".MainActivity"
	android:label="@string/app_name">
	
	<intent-filter>
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />
		
		<data android:scheme="iamporttest" /> <!-- 예시로 iamporttest로 설정. 앱을 표현하는 고유의 scheme을 사용하세요 -->
	</intent-filter>
	
</activity>
```

# 3.2 iOS  

아래의 github repository에서 KCP iOS 연동을 위한 샘플 프로젝트 확인이 가능합니다. 

[Objective-c용 KCP iOS 샘플](https://github.com/iamport/iamport-kcp-ios)

WebView내 웹페이지에서 호출되어야하는 javascript코드는 안드로이드와 동일합니다.  

```javascript
IMP.init('imp33886024'); //아임포트 회원가입 후 발급된 가맹점 고유 코드를 사용해주세요. 예시는 KCP공식 아임포트 데모 계정입니다.

IMP.request_pay({
    pg : 'html5_inicis',
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

### 3.2.1 WebView에서 외부 앱 호출을 위한 white-list 정의  

iOS보안정책에 의해 외부 호출될 scheme 을 `info.plist`에 나열해야 외부 앱 실행을 묻는 dialog가 나타나게 됩니다.  

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
	<string>payco</string> <!-- 페이코 -->
</array>
```

### 3.2.2 앱에 대한 URL scheme 정의  

외부앱을 통한 인증/결제 후 WebView가 실행되던 앱으로 복귀하기 위해서는 다음과 같이 앱에 대한 URL scheme을 정의해야 합니다.  
`IMP.request_pay(param, callback)`호출 시 `param.app_scheme`에 지정한 것과 동일한 값으로 URL Scheme을 정의해야 합니다.  

![Xcode Capture](screenshot/nice_xcode_scheme.png)


# 4. 에스크로 결제 연동  
아임포트가 지원하는 모든 PG사는 `escrow : true`옵션을 주면 에스크로 결제 모드로 동작하게 되어있습니다.  

```javascript
IMP.request_pay({
    pg : 'kcp', //웹표준 결제창 지원
    escrow : true, //에스크로 결제인 경우 필요
    pay_method : 'card', //card(신용카드), samsung(삼성페이), trans(실시간계좌이체), vbank(가상계좌), phone(휴대폰소액결제)
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
})
```

단, KCP 의 경우에는 상품별 부분배송의 경우를 고려하여 상품관련 정보를 추가적으로 전달해야할 필요가 있습니다.  
공통 매뉴얼에는 정의되어있지 않지만 `kcpProducts`라는 파라메터를 활용해 전달해주셔야 합니다.  

```javascript
IMP.request_pay({
    pg : 'kcp', //웹표준 결제창 지원
    escrow : true, //에스크로 결제인 경우 필요
    kcpProducts : [
    	{
			"orderNumber" : "xxxx",
			"name" : "상품A",
			"quantity" : 3,
			"amount" : 1000
		},
		{
			"orderNumber" : "yyyy",
			"name" : "상품B",
			"quantity" : 2,
			"amount" : 3000
		}
	],
    pay_method : 'card', //card(신용카드), samsung(삼성페이), trans(실시간계좌이체), vbank(가상계좌), phone(휴대폰소액결제)
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
})
```

`kcpProducts`는 array of object 형태이며, object는 다음과 같은 4개의 속성을 반드시 포함해야 합니다.  

- orderNumber : 상품주문번호
- name : 상품명
- quantity : 수량
- amount : 상품 가격

위 정보는 다른 아임포트 결제 파라메터와 비교검증하지는 않지만 (kcpProducts.amount 의 합계와 IMP.request_pay()의 amount는 관련이 없습니다.) 누락되면 에스크로 결제가 진행되지 않습니다.  


# 5. PAYCO 허브형 연동  

KCP와 계약된 상태에서도 PAYCO결제창을 연동하는 것이 가능합니다.(영업적으로 사전 논의필요) 이 경우, 신용카드 / 계좌이체 등과 마찬가지로 KCP를 통해 정산받으시는 것은 동일하지만 결제단계에서 구매자에게 제공되는 결제창이 PAYCO 연동시 나타나는 결제창이 제공되도록 하는 방법입니다.(이를 PAYCO 허브형이라고 지칭합니다)  

`pay_method : payco` 만 지정하면 PAYCO결제창이 나타나게 됩니다. (사전 협의된 후에 동작) 
  
```javascript
IMP.request_pay({
    pg : 'kcp', //웹표준 결제창 지원
    escrow : true, //에스크로 결제인 경우 필요
    pay_method : 'payco', //card(신용카드), samsung(삼성페이), trans(실시간계좌이체), vbank(가상계좌), phone(휴대폰소액결제), payco(허브형 PAYCO)
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
})
```