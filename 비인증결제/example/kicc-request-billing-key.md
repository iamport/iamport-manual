# KICC 정기결제(빌링) 연동 가이드 `결제창`

:globe_with_meridians: [EN](/en/Subscription/sample/kicc-request-billing-key.md)

KICC의 웹표준 결제창/모바일 결제창을 통해서 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다.

ℹ️ 자세한 내용은 [일반결제창으로 정기결제 연동하기](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-b)를 참고하세요.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/98b6d582-0dd2-43e0-95bd-037c5e05b416" target="_blank">KICC 정기결제 테스트 모드 설정</a> 페이지를 참고하여 PG 설정을 합니다.

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다.

ℹ️ 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 
	- 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용됩니다.
    - KICC에서 발급받은 상점아이디가 하나인 경우에는 `kicc`를, 여러개인 경우에는 `kicc.{PG상점아이디}`를 입력합니다.
- `customer_uid`: 빌링키 등록을 위해서 지정해야 합니다.
- `amount` : 결제창에 표시될 금액으로 실제 승인은 이루어지지 않습니다. 빌링키 발급과 함께 최초 결제를 하려면, 결제창에 금액이 표시되도록 금액을 지정하고 발급받은 [빌링키로 결제 요청](#request-pay)을 합니다.

```javascript
IMP.request_pay({
   	pg : 'kicc.빌링상점아이디',
	pay_method : 'card', // 'card'만 지원됩니다.
	merchant_uid: "order_monthly_0001", // 상점에서 관리하는 주문 번호
	name : '최초인증결제',
	amount : 1000, // 결제창에 표시될 금액. 실제 승인이 이뤄지지는 않습니다.
	customer_uid : 'your-customer-unique-id', // 필수 입력.
	buyer_email : 'iamport@siot.do',
	buyer_name : '아임포트',
	buyer_tel : '02-1234-1234',
	m_redirect_url : "https://yourservice.com/mobile/billing/landing" //모바일에서 빌링키 발급 후 이동할 URL
}, function(rsp) {
	if ( rsp.success ) {
		alert('빌링키 발급 성공');
	} else {
		alert('빌링키 발급 실패');
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
