# 알리페이 일반결제 연동 가이드

본 문서는 알리페이 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > 리디렉션 방식](../README.md#redirect)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

> 알리페이 결제 서비스 제공을 위해서는 국내 대행사인 **나이스페이먼츠와 계약**이 필요합니다.   
> USD 화폐단위가 기본 제공되며 KRW 추가 사용을 위해서는 **나이스페이먼츠와 사전 논의**가 필요합니다.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/f73c2a59-2fc3-4ea5-9765-6ff341e48916" target="_blank">알리페이 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 알리페이 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `alipay`로 지정합니다.
- `pay_method` : 호출 시 선택된 값은 무시되며, 페이코 결제창에서 선택한 옵션으로 설정됩니다.
- `currency` : 기본값으로 `USD`가 적용됩니다. (USD / KRW 지원)

```javascript
IMP.request_pay({
    pg : 'alipay',
    pay_method : 'card', // 생략 가능
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 1.2, //USD 1.2 달러 결제
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
});
```


