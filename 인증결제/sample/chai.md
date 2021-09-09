# 차이 일반결제 연동 가이드

본 문서는 차이 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > 리디렉션 방식](../README.md#redirect)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/97a21fe1-3e5f-4e77-9de1-38e55ebe73b1" target="_blank">차이 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 차이 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 
    - 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용됩니다.
	- 차이에서 발급받은 상점아이디가 하나인 경우에는 `chai`를, 여러개인 경우에는 `chai.{public_key}`를 입력합니다.
- `pay_method` : 'trans'만 가능합니다.

```javascript
IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
});
```


