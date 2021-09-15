# 다날 일반결제 연동 가이드

본 문서는 다날 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > Callback 방식](../README.md#callback)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/9865a7f8-4e20-4fe1-89bd-413beae49982" target="_blank">다날 일반결제 테스트 모드 설정</a> 페이지에서 결제수단에 따라 **1) 일반결제** 및/또는 **2) 휴대폰 소액결제** 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 다날 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 결제 수단에 따라 다음과 같이 설정합니다.
	- 신용카드/계좌이체/가상계좌 : `danal_tpay`
	- 휴대폰 소액결제 : `danal`
- `pay_method` : card(신용카드), trans(실시간계좌이체), vbank(가상계좌), 또는 phone(휴대폰소액결제)
- `buyer_tel`: 필수 입력 (미설정 시 다날 결제창에서 오류 발생 가능)  

### 2.1. 신용카드/계좌이체 결제창 호출 예제  

```javascript
IMP.request_pay({
    pg : 'danal_tpay',
    pay_method : 'card',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
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
### 2.2. 가상계좌 결제창 호출 예제  

- `biz_num`: `사업자등록번호 10자리` 필수 입력 (미설정 시 다날 결제창에서 오류 발생 가능)

```javascript
IMP.request_pay({
    pg : 'danal_tpay',
    pay_method : 'vbank',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    /* ...중략... */
    biz_num : '1234567890'   //필수 입력
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

