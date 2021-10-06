# 세틀뱅크 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/sample/settle.md)  

본 문서는 세틀뱅크 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기](../README.md#pc-mobile)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

⚠️ PC 결재 시, 팝업차단을 회피하기 위해서 아임포트 JavaScript SDK 1.1.7 이상 버전 사용을 권장합니다.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/4c37249c-be5e-45f9-84a2-769577d88736" target="_blank">세틀뱅크 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 세틀뱅크 결제창을 호출합니다.  

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.  

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `settle`로 지정합니다.
- `pay_method` : card(신용카드), trans(실시간계좌이체), vbank(가상계좌), 또는 phone(휴대폰소액결제)
- `company`: 가상계좌 결제 시, 예금주명, 회사명 혹은 서비스명을 입력합니다(은행 측 권고 사항).
- `buyer_tel`: 필수 입력 (미설정 시 세틀뱅크 결제창에서 오류 발생 가능)

```javascript
IMP.request_pay({
    pg : 'settle',
    pay_method : 'card',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    company : '아임포트', // 가상계좌 발급시 권고사항
    buyer_postcode : '123-456'
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

