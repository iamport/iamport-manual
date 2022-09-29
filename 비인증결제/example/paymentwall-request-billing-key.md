# 페이먼트월 정기결제(빌링) 연동 가이드 `결제창`

:globe_with_meridians: [EN](/en/Subscription/sample/paymentwall-request-billing-key.md)

페이먼트월의 웹표준 결제창/모바일 결제창을 통해서 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다.

ℹ️  자세한 내용은 [일반결제창으로 정기결제 연동하기](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-b)를 참고하세요.

ℹ️  페이먼트월과 별도 협의된 가맹점은 결제창/REST API를 사용하여 정기결제(빌링)를 연동할 수 있습니다. 해당 내용은 [페이먼트월 정기결제(빌링) 연동 가이드 (REST API 방식)](/비인증결제/example/paymentwall-api-billing-key.md)을 참고하세요.

## 1. PG 설정하기

<a href="https://chaifinance.notion.site/5aa0546574c94f159a7f040585af7359" target="_blank">페이먼트월 정기결제 테스트 모드 설정</a> 페이지의 **1) 결제창 방식**의 내용를 참고하여 PG 설정을 합니다.

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다.

ℹ️ 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

PC와 모바일 환경 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 결과를 받아볼수 있습니다.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `paymentwall`로 지정합니다.
- `customer_uid` : 빌링키 등록을 위해서 지정해야 합니다.
- `amount` : 빌링키 발급과 최초 결제 승인이 됩니다.
```javascript
IMP.request_pay({
    pg : 'paymentwall',
    pay_method : 'card',
    merchant_uid: 'order_no_0001', //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 50,
    buyer_email : 'iamport@example.com',   //필수파라미터
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    currency : 'USD',
    language : 'en',
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}', // 예: https://www.my-service.com/payments/complete/mobile,
    customer_uid : 'customer_uid', //빌링키 발급 시 필수파라미터
    bypass: {
      widget_code: "t3_1",  //터미날3 인경우 해당 파라미터 설정, 미 설정시 Defualt(일반) 결제창 활성화
    }
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

<a name="request-pay" />

## 3. 빌링키로 결제 요청하기  

빌링키 발급이 성공하면 빌링키는 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 저장됩니다. 보안상의 이유로 서버는 빌링키에 직접 접근할 수 없기 때문에 `customer_uid`를 이용해서 재결제([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) REST API를 다음과 같이 호출합니다.
이때, 구매자의 이메일 주소는 빌링키 발급시 입력했던 이메일과 동일해야 정상적으로 결제가 처리됩니다.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000, "buyer_email" : "iamport@example.com"}' \
     https://api.iamport.kr/subscribe/payments/again
```
