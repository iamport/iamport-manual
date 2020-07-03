# 1. PG 설정  

## 1.1 테스트모드 설정  
![블루월넛-아임포트 시스템 설정](screenshot/bluewalnut.png)

- PG사 : 블루월넛 선택 후, 테스트모드 설정 ON상태로 두고 저장합니다.

## 1.2 상용모드 설정  

1. [아임포트 PG가입 페이지](http://www.iamport.kr/pg)에서 블루월넛 가입 신청을 합니다. 
2. PG사 : 블루월넛 선택 후, 테스트모드 설정 OFF상태로 둡니다. 
3. 계약 후 블루월넛에서 발급받은 PG상점아이디(MID) / KEY 를 입력 후 저장합니다. 


# 2. PC, 모바일 브라우저 연동  

블루월넛의 경우 레이어를 통해 결제창이 제공되며 페이지 리디렉션이 없어 PC, 모바일 브라우저 환경에서 동일한 소스코드를 적용할 수 있습니다.  

페이지 리디렉션이 없기 때문에 PC, 모바일 모두 `IMP.request_pay(param, callback)`의 callback 함수를 활용할 수 있습니다.  

- 블루월넛을 의미하는 `pg` 구분자는 `bluewalnut` 입니다. 
- 블루월넛을 "기본PG사"로 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. 
- 블루월넛을 "추가PG사"로 사용하시는 경우에는 `pg`파라메터를 `bluewalnut` 또는 `bluewalnut.{블루월넛 상점아이디}` 로 설정하시면 됩니다. [pg파라메터 사용 상세 가이드](https://docs.iamport.kr/tech/pg-parameter)    


```javascript
IMP.request_pay({
    pg : 'bluewalnut',
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