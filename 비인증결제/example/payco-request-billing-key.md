# 페이코 정기결제(빌링) 연동 가이드 `결제창`

:globe_with_meridians: [EN](./en/payco-request-billing-key.md)

페이코의 간편결제창을 통해서 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다.<Br />

ℹ️ 자세한 내용은 [일반결제창으로 정기결제 연동하기](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-b)를 참고하세요.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/d6601ff5-49ec-45a6-8bfb-79752e700942" target="_blank">페이코 정기결제 테스트 모드 설정</a> 페이지를 참고하여 PG 설정을 합니다.

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/tech/imp#request_pay)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다. 

ℹ️ 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `payco`로 지정합니다.
- `customer_uid` : 빌링키 등록을 위해서 지정해야 합니다.
- `amount` : 결제창에 표시될 금액으로 실제 승인은 이루어지지 않습니다. 빌링키 발급과 함께 최초 결제를 하려면, 결제창에 금액이 표시되도록 금액을 지정하고 발급받은 [빌링키로 결제 요청](#request-pay)을 합니다.

```javascript
IMP.request_pay({
    pg : 'payco', 
	pay_method : 'card', // 결제창에서 선택된 값은 무시되며, 결제 완료 시점에 설정됩니다.
	merchant_uid : 'merchant_' + new Date().getTime(),
	name : 'PAYCO 자동결제 등록',
	amount : 1000, // 결제창에 표시될 금액. 실제 승인이 이뤄지지는 않습니다.
	customer_uid : 'your-customer-unique-id', // 필수 입력.
	buyer_email : 'iamport@siot.do',
	buyer_name : '아임포트',
	buyer_tel : '02-1234-1234'
}, function(rsp) {
	if ( rsp.success ) {
		alert('빌링키 발급(자동결제 등록) 성공');
	} else {
		alert('빌링키 발급(자동결제 등록) 실패');
	}
});
```

<a name="request-pay" />

## 3. 빌링키로 결제 요청하기

빌링키 발급이 성공하면 빌링키는 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 저장됩니다. 보안상의 이유로 서버는 빌링키에 직접 접근할 수 없기 때문에 `customer_uid`를 이용해서 재결제([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) REST API를 다음과 같이 호출합니다.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again	 
```
