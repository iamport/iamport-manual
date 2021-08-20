# 차이 정기결제(빌링) 연동 가이드 `결제창`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/en/chai-request-billing-key.md">EN</a>

차이의 간편결제창을 통해서 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다.<Br />

ℹ️ 자세한 내용은 [일반결제창으로 정기결제 연동하기](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-b)를 참고하세요.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/195ae9aa-d862-4fb6-a637-3065c3dae1e7" target="_blank">차이 정기결제 테스트 모드 설정</a> 페이지를 참고하여 PG 설정을 합니다.

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/tech/imp#request_pay)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다.

ℹ️ 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 차이에서 발급받은 상점아이디가 하나인 경우에는 `pg: 'chai'`를, 여러개인 경우에는 `pg: 'chai.{public_key}'`를 입력합니다.
- `amount` : 결제창에 표시될 금액으로 실제 승인은 이루어지지 않습니다. 빌링키 발급과 함께 최초 결제를 하려면, 결제창에 금액이 표시되도록 금액을 지정하고 발급받은 [빌링키로 결제 요청](#request-pay)을 합니다.

```javascript
IMP.init('{가맹점 식별코드}'); // 예: imp37739582(차이 공식 데모 계정용 가맹점 식별코드)

IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans', // 차이는 계좌기반의 서비스이므로 trans만 입력 가능
    merchant_uid : '{고유 주문번호}', // 예: issue_billingkey_monthly_0001
    name : '주문명: 결제 테스트',
    amount : 0, // 결제창에 표시될 금액. 결제승인을 하지 않음
    customer_uid : '{카드(빌링키)와 1:1로 대응하는 값}', // 필수 입력 (예: gildong_0001_1234)
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자 이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
}, function(rsp) {
    if ( !rsp.success ) {
    	// 차이 결제 시작 페이지로 리디렉션되기 전에 오류가 난 경우
        var msg = '오류로 인하여 요청을 처리하지 못했습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

<a name="request-pay" />

## 3. 빌링키로 결제 요청하기

빌링키 발급이 성공하면 빌링키는 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 저장됩니다. 보안상의 이유로 서버는 빌링키에 직접 접근할 수 없기 때문에 `customer_uid`를 이용해서 재결제([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) REST API를 다음과 같이 호출합니다.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"merchant_xxxxxxxx", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```
