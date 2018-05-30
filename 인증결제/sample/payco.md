# 1. PC, 모바일 브라우저 연동  


PAYCO의 경우 제공되는 결제창을 활용했을 때 서비스의 기본 페이지로부터 이동을 할 필요가 없어 PC, 모바일 브라우저 환경에서 동일한 소스코드를 적용할 수 있습니다.  

페이지 이동이 없기 때문에 `IMP.request_pay(param, callback)`의 callback 함수를 활용할 수 있습니다.  

- PAYCO를 "기본PG사"로 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. 
- PAYCO의 경우 `pay_method`파라메터로 특정 결제수단 지정이 불가능하기 때문에 `card`, `trans`, `vbank`, `phone` 중 아무 것이든 사용 가능합니다. (결제가 완료된 시점에 pay_method를 판단하여 변경합니다.)  


```javascript
IMP.request_pay({
    pg : 'payco',
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

## 1.1 팝업 BLOCK 이슈  
PAYCO결제는 PC/모바일에서 팝업창(새창)을 통해 진행되므로, 사용자의 브라우저 보안정책에 의해 팝업이 차단될 수 있습니다.  
팝업이 차단되거나 사용자 Confirm을 회피하기 위해서는 `IMP.request_pay(param, callbak)`호출은 `onClick`과 같이 사용자 액션에 대한 핸들러에서 처리되어야 합니다.  

