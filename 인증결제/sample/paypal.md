# 페이팔 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/sample/paypal.md)  

본 문서는 페이팔 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > 리디렉션 방식](../README.md#redirect)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/458b899c-1058-4055-8b73-a171b5354c2e" target="_blank">페이팔 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 페이팔 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 `m_redirect_url`로 리디렉션됩니다. 

ℹ️ 한국 사업자의 경우 **PayPal Standard** 와 **Express Checkout** 2가지 결제 방법을 사용할 수 있습니다. 아임포트에서는 사용자 이탈률이 적고 결제 데이터 누락 오류가 없는 **Express Checkout**을 적용합니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `paypal`으로 지정합니다.
- `pay_method` : card만 가능합니다.
- `currency` : 지원 가능한 모든 통화는 [페이팔 공식 문서](https://developer.paypal.com/docs/api/reference/currency-codes/#paypal-account-payments)를 참고해주세요.

```javascript
IMP.request_pay({
    pg : 'paypal',
    pay_method : 'card',
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 14.20,
    currency : 'USD', // 기본값: USD(원화 KRW는 페이팔 정책으로 인해 지원하지 않음)
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete
});
```

 
