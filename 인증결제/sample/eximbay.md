# 엑심베이 일반결제 연동 가이드

본 문서는 엑심베이 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > Callback 방식](../README.md#callback)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview) 

## 1. PG 설정하기

<a href="https://guide.iamport.kr/cdf5c3aa-2529-40ed-bc5a-9daba183c655" target="_blank">엑심베이 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 엑심베이 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `eximbay`로 지정합니다.
- `pay_method` : 다음 결제수단 중 선택합니다.
	- 위챗 : wechat   
	- 알리페이 : alipay   
	- 해외카드 : card   
	- 유니온페이 : unionpay   
	- 텐페이 : tenpay
	- 일본 편의점결제(eContext) :  econtext
- `language` : 언어
	- 한국어 : ko    
	- 영어 : en    
	- 중국어 : zh    
	- 일본어 : jp
- `currency` : 통화(화폐단위), 엑심베이가 지원하는 currency 사용이 모두 가능      
	- KRW      
	- USD      
	- EUR      
	- GBP      
	- JPY      
	- THB      
	- SGD      
	- RUB      
	- HKD      
	- CAD      
	- AUD
- `buyer_tel`: 필수 입력 (미설정 시 엑심베이 결제창에서 오류 발생 가능)

```javascript
IMP.request_pay({
    pg : 'eximbay',
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
 
