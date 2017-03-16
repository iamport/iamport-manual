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
    buyer_postcode : '123-456',
    proxyPath : '/path1/path2'
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
PC에서 PAYCO결제창 호출을 위해 `IMP.request_pay(param, callback)`가 호출되었을 때, 브라우저에서 PAYCO도메인에 대한 팝업이 차단되어 팝업 허용 후 새로고침해야하는 문제점이 있었습니다.  
이러한 불편함을 해소하기위해서는 아임포트 PROXY기능을 활용해주셔야 합니다.  

### 1.1.1 PROXY페이지 준비  

```html
<script type="text/javascript" src="https://service.iamport.kr/js/iamport.payment.proxy.js"></script>
```
가 추가된 HTML페이지를 생성합니다(다른 js는 필요하지 않으며, 위 스크립트 태그만 추가되면 됩니다.)  
**주의 : 만들어진 페이지는 결제를 진행하는 웹서비스와 동일한 도메인에서 접근이 가능해야 합니다.**  

### 1.1.2 proxyPath 설정  
1.1.1에서 만든 웹 페이지의 **path**을 파라메터로 전달하시면 됩니다. 
`IMP.request_pay(param, callback)` 호출 시 `param.proxyPath` 파라메터를 활용하면 되고, `/`로 시작하는 path정보만 포함되어야 합니다.  
가령, 1.1.1에서 제작한 PROXY페이지의 URL이 `http://www.my-commerce.com/path1/path2` 라면 `param.proxyPath : '/path1/path2'` 를 지정하시면 됩니다.  

*PROXY기능은 iamport.payment-1.1.4.js에서만 현재 제공되고 있습니다. (1.1.3, 1.1.2버전에도 조만간 적용될 예정입니다)*  

