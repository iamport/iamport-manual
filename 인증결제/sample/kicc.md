# 1. PC 연동  

`IMP.request_pay(param, callback)`의 callback 함수를 활용하여 결제완료 통지를 받을 수 있습니다.  

- KICC(한국정보통신)을 "기본PG사"로 설정해 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. (관리자 페이지에 설정된 정보를 기본으로 사용합니다)  
- 복수개의 PG설정을 해두신 경우 KICC결제창을 호출하시려면 `pg`파라메터를 `kicc`로 설정해주세요.  


```javascript
IMP.request_pay({
    pg : 'kicc', //아임포트 관리자에서 기본PG만 설정하신 경우는 생략 가능
    pay_method : 'card', //card(신용카드), trans(실시간계좌이체), vbank(가상계좌), phone(휴대폰소액결제)
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //누락되면 카드사 인증에 실패할 수 있으니 기입해주세요
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

KICC 모바일 브라우저 결제에서는 페이지 이동이 자동으로 이뤄지기 때문에 `IMP.request_pay(param)`호출을 하셔야하고, PC와 달리 `callback` 함수를 사용하실 수 없습니다.  
대신, `param`에 `m_redirect_url`파라메터를 설정해 결제완료를 인지하실 수 있습니다.  

```javascript
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

# 3. 에스크로 결제 연동  
아임포트가 지원하는 모든 PG사는 `escrow : true`옵션을 주면 에스크로 결제 모드로 동작하게 되어있습니다.  
KICC의 경우 현금성 결제수단(실시간계좌이체, 가상계좌)에 한하여 에스크로 결제모드가 지원되며, KICC의 에스크로 결제 정책상, `buyer_name`, `buyer_email`, `buyer_tel` 파라메터는 반드시 지정되어야 하며 빈 문자열이면 안됩니다.  

```javascript
IMP.request_pay({
    pg : 'kicc', //웹표준 결제창 지원
    escrow : true, //에스크로 결제인 경우 필요
    pay_method : 'vbank', //trans(실시간계좌이체), vbank(가상계좌)
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

단, KICC 의 경우에는 상품별 부분배송의 경우를 고려하여 상품관련 정보를 추가적으로 전달해야할 필요가 있습니다.  
공통 매뉴얼에는 정의되어있지 않지만 `kiccProducts`라는 파라메터를 활용해 전달해주셔야 합니다.  

```javascript
IMP.request_pay({
    pg : 'kicc', //웹표준 결제창 지원
    escrow : true, //에스크로 결제인 경우 필요
    kiccProducts : [
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
    pay_method : 'vbank', //trans(실시간계좌이체), vbank(가상계좌)
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

`kiccProducts`는 array of object 형태이며, object는 다음과 같은 4개의 속성을 반드시 포함해야 합니다.  

- orderNumber : 상품주문번호
- name : 상품명
- quantity : 수량
- amount : 상품 가격

위 정보는 다른 아임포트 결제 파라메터와 비교검증하지는 않지만 (kiccProducts.amount 의 합계와 IMP.request_pay()의 amount는 관련이 없습니다.) 
kiccProducts 파라메터는 누락되면 에스크로 결제가 진행되지 않습니다.    
