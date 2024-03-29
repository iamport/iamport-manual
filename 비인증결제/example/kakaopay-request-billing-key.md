# 카카오페이 정기결제(빌링) 연동 가이드 `결제창`

:globe_with_meridians: [EN](/en/Subscription/sample/kakaopay-request-billing-key.md)

카카오페이의 간편결제창을 통해서 빌링키 발급과 최초 결제를 같이 요청하거나, 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다.<Br />

ℹ️ 자세한 내용은 [일반결제창으로 정기결제 연동하기](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-b)를 참고하세요.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/0f2bec60-0a7b-4626-9d72-7af82740ceb2" target="_blank">카카오페이 정기결제 테스트 모드 설정</a> 페이지를 참고하여 PG 설정을 합니다.

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다.

ℹ️ 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `kakaopay`로 지정합니다.
- `customer_uid` : 빌링키 등록을 위해서 지정해야 합니다.
- `amount` : 빌링키 발급만 하는 경우 "0"으로 지정하고, 빌링키 발급과 최초 결제를 같이 요청하는 경우 가격을 지정합니다.  

### 2.1 빌링키 발급만 요청하기(amount : 0)

```javascript
IMP.request_pay({
	merchant_uid: "order_monthly_0001", // 상점에서 관리하는 주문 번호
	name : '최초인증결제',
	amount : 0, // 빌링키 발급만 진행하며 결제승인을 하지 않습니다.
	customer_uid : 'your-customer-unique-id', // 필수 입력
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

### 2.2 빌링키 발급 및 최초 결제 요청하기(amount : 가격지정)

- `pay_method` : 결제창에서 선택된 값은 무시되며, 카카오페이 앱에서 신용카드와 카카오머니 중 선택한 옵션으로 설정되며 재결제 시 선택이 유지됩니다.

```javascript
IMP.request_pay({
	pay_method : 'card', // 기능 없음.
	merchant_uid: "order_monthly_0001", // 상점에서 관리하는 주문 번호
	name : '최초인증결제',
	amount : 1004, // 빌링키 발급과 함께 1,004원 결제승인을 시도합니다.
	customer_uid : 'your-customer-unique-id', // 필수 입력
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

## 3. 빌링키로 결제 요청하기

빌링키 발급이 성공하면 빌링키는 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 저장됩니다. 보안상의 이유로 서버는 빌링키에 직접 접근할 수 없기 때문에 `customer_uid`를 이용해서 재결제([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) REST API를 다음과 같이 호출합니다.<Br />  

Kakao Pay는 월별 최대 재결제 횟수 제한이 있습니다.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```

