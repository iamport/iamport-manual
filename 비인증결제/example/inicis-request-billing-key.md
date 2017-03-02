# KG이니시스 빌링키 발급
빌링키 발급을 위해 카드사에 카드정보를 전달할 때, KG이니시스의 웹표준 결제창/모바일 결제창을 통해 직접 PG사로 전달하는 방식만 제공합니다.  
(`/subscribe/payments/onetime`, `/subscribe/customers/{customer_uid}` API를 사용할 수 없음)


## 1. PG설정  
KG이니시스 웹표준 결제창을 통해 빌링키 발급이 진행되어야 하므로 아임포트 관리자 페이지의 시스템 설정 > PG설정(인증방식결제)에서 설정합니다.  
![KG이니시스 빌링 설정](../screenshot/inicis-setting.png)

- KG이니시스(웹표준결제창) 선택  
- Sandbox를 Off로 변경  
- PG상점아이디에 `INIBillTst` 입력  
- 웹표준결제 signKey에 `SU5JTElURV9UUklQTEVERVNfS0VZU1RS` 입력  
- 빌링용 merchantKey에 `b09LVzhuTGZVaEY1WmJoQnZzdXpRdz09` 입력  

`param.customer_uid`를 지정하셔서 기존처럼 결제창 호출하시면 이니시스 빌링결제창이 나타납니다. 
PC/모바일 모두 `IMP.request_pay(param, callback)`호출 후 callback으로 실행되니 이 점 참고부탁드립니다.(`m_redirect_url`이 사용되지 않음)

## 2. 빌링키 발급을 위한 결제창 호출
인증방식의 결제를 위해 `iamport.payment.js`의 `IMP.request_pay(param, callback)` 와 동일한 인터페이스를 사용합니다.  
*(파라메터 등의 정보는 [인증결제](https://github.com/iamport/iamport-manual/tree/master/%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C) 매뉴얼 내용을 참고해주세요.)*  

*알려진 버그 : KG이니시스 웹표준 결제창 특성상, amount는 반드시 0보다 큰 숫자를 지정해야만 합니다. 이 때문에 빌링키 발급 결제창에 구매금액이 표시가 되나, 실제로 결제가 되지는 않으므로 1원 정도만 지정해주세요.*  
*(모바일 결제에서는 이와 같은 문제가 없습니다.)*

### 빌링키 발급 결제창 호출  

```javascript
IMP.request_pay({
	pay_method : 'card', // 'card'만 지원됩니다.
	merchant_uid : 'merchant_' + new Date().getTime(),
	name : '최초인증결제',
	amount : 1, // 결제창에 1원이 표시되나, 실제로 결제승인이 이루어지지는 않습니다.(모바일에서는 가격도 표시되지 않음)
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