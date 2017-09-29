# 모빌리언스-휴대폰 빌링키 발급
모빌리언스-휴대폰 결제창을 통해 최초 결제가 이루어짐과 동시에 빌링키가 발급됩니다.  
(`/subscribe/customers/{customer_uid}` API 및 `/subscribe/payments/onetime`를 사용할 수 없음)  
휴대폰 정기결제의 특성상 최초 결제된 금액과 동일한 날짜에 매월 재결제가 이루어져야 합니다(날짜 오차 5일 이내)  


## 1. PG설정  
모빌리언스-휴대폰 결제창을 통해 결제와 빌링키 발급이 진행되어야 하므로 아임포트 관리자 페이지의 시스템 설정 > PG설정(인증방식결제)에서 설정합니다.  
*(모빌리언스 일반 휴대폰결제설정과 설정방법은 동일합니다)*

- 모빌리언스-휴대폰소액결제 선택
- Sandbox를 On으로 변경



## 2. 빌링키 발급을 위한 결제창 호출
인증방식의 결제를 위해 `iamport.payment.js`의 `IMP.request_pay(param, callback)` 와 동일한 인터페이스를 사용합니다.  
*(모빌리언스 결제모듈은 PC와 모바일의 차이점이 없으므로 아래의 코드를 PC/모바일에서 모두 사용 가능합니다.)*  

**`customer_uid` 파라메터의 유무로 모빌리언스 일반 휴대폰결제와 빌링키 발급용 최초 결제를 구분하게 됩니다.**



```javascript
IMP.request_pay({
	pay_method : 'phone', // 'phone'만 지원됩니다.
	merchant_uid : 'merchant_' + new Date().getTime(),
	name : '최초인증결제',
	amount : 10000, // 빌링키 발급과 동시에 10,000원 결제승인이 이루어집니다. 다음 정기결제부터 10,000원 결제가 이뤄져야합니다. 
	customer_uid : 'your-customer-unique-id', //customer_uid 파라메터가 있어야 빌링키 발급을 시도합니다.
	buyer_email : 'iamport@siot.do',
	buyer_name : '아임포트',
	buyer_tel : '02-1234-1234'
}, function(rsp) {
	if ( rsp.success ) {
		alert('빌링키 발급 성공');
	} else {
		alert('빌링키 발급 실패');
	}
});
```


## 3. 발급된 빌링키로 결제요청  
빌링키 발급이 성공적으로 이루어지면, 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 보관됩니다.
때문에, `customer_uid`를 전달하면 발급된 빌링키를 찾아 결제승인 요청을 진행하게 됩니다.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```