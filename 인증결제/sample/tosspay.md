# 토스간편결제 연동 가이드

본 문서는 토스간편결제 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기 > 리디렉션 방식](../README.md#redirect)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href ='https://chaifinance.notion.site/9f14768770bd486f92c10fde5497216a' target='blank'>토스간편결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 차이 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 
    - 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용됩니다.
    - Toss 간편결제는 상점아이디라는 컨셉이 없고 apiKey 만 존재합니다. (단, apiKey 는 유출되면 안되는 민감정보)
    - 통상적으로 모든 PG가 상점아이디 개념을 지니고 있기 때문에 아임포트에서는 pg_id 로 식별하고 있습니다.
    - 이 정책을 유지하기 위해 Toss 간편결제는 다음과 같이 2가지로 pg_id 를 자동 생성합니다.
    
          * tosstest : 테스트모드의 토스간편결제 pg_id
          * tosspay : 상용모드의 토스간편결제 pg_id
     
- `pay_method` : 'card' or 'trans'만 가능합니다.

### 토스간편결제는 Javascript SDK Version 1.2.0 이상 버전을 사용해야 합니다.

#### 연동예시
```javascript
IMP.request_pay({
    pg : 'tosspay',
    pay_method : 'card',
    merchant_uid: "order_no_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 1004,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```


## 3. Toss 간편결제 동작 방법
   * PC결제 : 팝업창을 통한 결제 진행
   * 모바일결제 : 페이지 리디렉션을 통한 결제 진행


## 4. 기타 정보

결제 후 저장되는 '승인'에 대한 extension 정보 ( 정산금액 관련 )
 
- paidPoint : 승인금액 중 토스머니 차감액
- discountedAmount : 승인금액 중 할인적용 금액
- paidAmount : 승인금액 중 주결제수단(신용카드/계좌이체) 승인금액

환불 후 저장되는 매 '환불' 건에 대한 extension 정보 ( 정산금액 관련 )
 
- refundedPoint : 해당환불금액 중 토스머니 환불액
- refundedDiscountAmount : 해당환불금액 중 취소된 할인금액
- refundedPaidAmount : 해당환불금액 중 주결제수단(신용카드/계좌이체) 환불금액

