# PaymentWall 일반결제 연동 가이드


본 문서는 PaymentWall 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기](../README.md#pc-mobile)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href='https://www.notion.so/chaifinance/Paymentwall-6e4ddcd60ec64324bcc91e7fbf896657'>PaymentWall 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.(수정필요)

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 PaymentWall 결제창을 호출합니다.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `paymentwall`로 지정합니다.
- `pay_method` : 
    - card(신용카드) 로 지정하여도 모든 결제수단이 활성화 됩니다.

```javascript
IMP.request_pay({
    pg : 'paymentwall',
    pay_method : 'card',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 50,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    currency : 'USD'
    language : 'en'
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile,
    bypass: {
      widget_code: "t3_1"  //터미날3 인경우 해당 파라미터 설정, 미 설정시 Defualt(일반) 결제창 활성화
    }
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

