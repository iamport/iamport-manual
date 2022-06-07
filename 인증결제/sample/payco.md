# 페이코 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/sample/payco.md)  

본 문서는 페이코 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > Callback 방식](../README.md#callback)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/09f252f7-f15e-4e1c-ad39-d526486b463b" target="_blank">페이코 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 페이코 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `payco`로 지정합니다.
- `pay_method` : 호출 시 선택된 값은 무시되며, 페이코 결제창에서 선택한 옵션으로 설정됩니다.
- `name` : 필수 입력 (미설정 시 페이코 결제창에서 오류 발생 가능)

```javascript
IMP.request_pay({
    pg : 'payco',
    pay_method : 'card', //생략 가능
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```  

