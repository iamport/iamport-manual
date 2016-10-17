#1. PC 연동
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

#2. 모바일 브라우저 연동

현재 KICC모듈 모바일 연동 준비 중입니다.  