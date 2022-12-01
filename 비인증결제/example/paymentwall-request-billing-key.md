# 페이먼트월 정기결제(빌링) 연동 가이드 `결제창`

:globe_with_meridians: [EN](/en/Subscription/sample/paymentwall-request-billing-key.md)

페이먼트월의 웹표준 결제창/모바일 결제창을 통해서 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다.

ℹ️  자세한 내용은 [일반결제창으로 정기결제 연동하기](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-b)를 참고하세요.


## 1. PG 설정하기

<a href="https://chaifinance.notion.site/5aa0546574c94f159a7f040585af7359" target="_blank">페이먼트월 정기결제 테스트 모드 설정</a> 페이지의 **1) 결제창 방식**의 내용를 참고하여 PG 설정을 합니다.

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 바로 빌링키를 발급하거나 빌링키 발급을 위한 결제창을 호출합니다.

ℹ️ 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

PC와 모바일 환경 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 결과를 받아볼수 있습니다.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `paymentwall`로 지정합니다.
- `customer_uid` : 빌링키 등록을 위해서 지정해야 합니다.
- `amount` : 빌링키 발급과 최초 결제 승인이 됩니다.
- `card.info` : 페이먼트월 결제창 호출 없이 빌링키 발급하려는 경우 전달합니다. (페이먼트월 [Brick](https://docs.paymentwall.com/integration/direct/brick-home) 기능이 활성화된 가맹점에 대해서만 사용가능합니다.)
  - `card.info.number` : 빌링키 발급 대상 카드번호 (dddd-dddd-dddd-dddd) 
  - `card.info.expiry` : 빌링키 발급 대상 카드 유효기간 (YYYY-MM)
  - `card.info.cvc` : 빌링키 발급 대상 카드 cvc

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
    card : { //페이먼트월 결제창 호출 없이 빌링키 발급하려는 경우 설정
      info: { 
        number: '4242-4242-4242-4242',
        expiry: '2026-09',
        cvc: '826'
      }
    },
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

> **주의사항**
> - 페이먼트월의 결제 처리는 핑백에 의해 비동기로 처리되고 있기 때문에 빌링키 발급 요청 이후 아임포트로부터 웹훅을 수신한 이후 부터 빌링키 결제 요청이 가능합니다.
> - 페이먼트월 결제창 호출 없이 빌링키를 발급하는 경우 `IMP.request_pay(param, callback)` 호출만으로 결제와 빌링키 발급이 이루어지기 때문에 여러번 호출되지 않도록 사용에 주의가 필요합니다.
