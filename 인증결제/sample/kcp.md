# NHN KCP 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/sample/kcp.md)  

본 문서는 NHN KCP 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기](../README.md#pc-mobile)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/bc4ded5d-c9f0-4f0e-bd1c-160859748fdd" target="_blank">NHN KCP 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 NHN KCP 결제창을 호출합니다.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `kcp`로 지정합니다.
- `pay_method` : 
    - card(신용카드), samsung(삼성페이), trans(실시간계좌이체), vbank(가상계좌), 또는 phone(휴대폰소액결제), payco(페이코 허브형)
    - payco: KCP를 통해서 결제수단을 PAYCO를 선택하고 PAYCO 결제창으로 진행하는 방식 ([NHN KCP PAYCO 허브형 신청안내](https://sir.kr/main/service/p_payco_hub.php)) 


```javascript
IMP.request_pay({
    pg : 'kcp',
    pay_method : 'card',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

### Escrow 결제

에스크로 결제요청 시 상품별 부분배송을 고려하여 상품관련 정보를 추가 파라미터(`kcpProducts`)로 전달해야 합니다.  
`kcpProducts`는 다음 4개의 필수 속성으로 구성된 객체배열입니다. 해당 `amount` 값은 결제 금액(`param.amount`) 값과 관계가 없으며 비교검증되지 않습니다.

- orderNumber : 상품주문번호
- name : 상품명
- quantity : 수량
- amount : 상품 가격

```javascript
IMP.request_pay({
    pg : 'kcp',
    escrow : true, //에스크로 결제인 경우 필요
    kcpProducts : [
    	{
			"orderNumber" : "xxxx",
			"name" : "상품A",
			"quantity" : 3,
			"amount" : 1000
		},
		{
			"orderNumber" : "yyyy",
			"name" : "상품B",
			"quantity" : 2,
			"amount" : 3000
		}
	],
    //* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
})
```

