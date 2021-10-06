# 카카오페이 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/sample/kakao.md)  

본 문서는 카카오페이 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > Callback 방식](../README.md#callback)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

> 2018년 5월부로 (주)LGCNS가 운영하던 카카오페이는 종료되며, (주) 카카오페이가 운영하는 카카오페이로 전환됩니다. LGCNS 사용자가 카카오페이 이관이 이뤄질 수 있도록 당분간 병행할 예정입니다.  
> 
> 코드는 모두 동일하나, pg\_provider 파라메터가 서로 다르니 주의바랍니다. 
> 
> 기존 카카오페이 - pg\_provider : kakao  
> 신규 카카오페이 - pg\_provider : kakaopay

## 1. PG 설정하기

<a href="https://guide.iamport.kr/51110e50-4c48-43bc-b0b5-a2161f4b5af8" target="_blank">카카오페이 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 카카오페이 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `kakaopay`로 지정합니다. 
- `pay_method` : 호출 시 선택된 값은 무시되며, 카카오페이 앱에서 신용카드와 카카오머니 중 선택한 옵션으로 설정됩니다. 


```javascript
IMP.request_pay({
    pg : 'kakaopay',
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

