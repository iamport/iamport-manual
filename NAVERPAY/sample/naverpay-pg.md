# 결제형 네이버페이 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/NAVERPAY/sample/naverpay-pg.md)  

결제형 네이버페이 일반결제는 일반 PG처럼 결제 수단 중 하나로 네이버페이를 선택하여 결제가 진행됩니다.  

본 문서는 네이버페이 관련 내용만 기술하므로 다음 일반결제 연동 정보를 참고하세요.  

- [PC/모바일 웹에서 PG 연동하기](/인증결제/README.md#pc-mobile)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/485c6da8-01d7-4900-bc05-76005e5477ba" target="_blank">네이버페이(주문형/결제형) 테스트 모드 설정</a> 페이지에서 **2) 결제형** 내용을 참고하여 PG 설정을 합니다.

## 2. 결제창 호출하기  

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 네이버페이 결제창을 호출합니다.

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 파라미터(`naverPopupMode`) 설정을 통해 팝업 방식(callback 호출) 또는 리디렉션 방식(`m_redirect_url`로 페이지 이동)으로 진행될 수 있습니다. [PC, 모바일 버전에서 진행 방식](#method)을 참고하세요.  

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `naverpay`로 지정합니다.
- `buyer_tel` : 필수 입력.
- `naverUseCfm` : 이용 완료일((yyyyMMdd 형식의 문자열로 결제 당일 또는 미래의 일자여야 함).
	-   상품 유형에 따라, 네이버페이-가맹점 간 필수값으로 계약되는 경우 입력합니다.  
- `name`: 주문명.
	- `naverProduts` 파라미터에 따라 네이버페이 내부적으로 `외 2개`의 표현이 자동생성되므로 `xxxx 외 2개` 대신 `naverProducts[0].name`(첫번째 상품명)으로 설정하길 권장합니다.
- `naverPopupMode` : 팝업 방식으로 진행 여부 (true/false).
	- `false`인 경우, 페이지 리디렉션 방식으로 진행되며 `m_redirect_url`을 설정해야 합니다.
- `m_redirect_url` : 리디렉션 방식으로 진행(`naverPopupMode`: false)할 경우 결제 완료 후 리디렉션 될 URL. 
- `purchaserName` : 구매자 이름, 결제 상품이 보험 및 위험 업종 등인 경우 필수 입력.
- `purchaserBirthday` : 구매자 생년월일(yyyyMMdd), 결제 상품이 보험 및 위험 업종 등인 경우 필수 입력.
- [naverProducts](#naverProducts) : 상품정보(필수 입력). 네이버페이 매뉴얼의 `productItems` 파라미터와 동일합니다.

```javascript
IMP.request_pay({
    pg : 'naverpay',
    merchant_uid: "order_no_0001", //상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 14000,
    tax_free : 4000, //면세공급가액(누락되면 0원으로 처리)
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
	naverUseCfm : '20201001', //이용완료일자
    naverPopupMode : true, //팝업모드 활성화
	m_redirect_url : "{결제 완료 후 리디렉션 될 URL}", //예 : http://yourservice.com/payments/complete
    purchaserName: "구매자이름",
    purchaserBirthday: "20151201",
    naverProducts : [{
      "categoryType": "BOOK",
			"categoryId": "GENERAL",
			"uid": "107922211",
			"name": "한국사",
			"payReferrer": "NAVER_BOOK",
			"sellerId": "sellerA",
			"count": 10
		},
		{
			"categoryType": "MUSIC",
			"categoryId": "CD",
			"uid": "299911002",
			"name": "러블리즈",
			"payReferrer": "NAVER_BOOK",
			"sellerId": "sellerB",
			"count": 1
		}]
}, function(rsp) { //팝업 방식으로 진행 또는 결제 프로세스 시작 전 오류가 발생할 경우 호출되는 callback
    if ( rsp.success ) {
    	//[1] 서버단에서 결제정보 조회를 위해 jQuery ajax로 imp_uid 전달하기
    	jQuery.ajax({
    		url: "/payments/complete", //cross-domain error가 발생하지 않도록 주의해주세요
    		type: 'POST',
    		dataType: 'json',
    		data: {
	    		imp_uid : rsp.imp_uid
	    		//기타 필요한 데이터가 있으면 추가 전달
    		}
    	}).done(function(data) {
    		//[2] 서버에서 REST API로 결제정보확인 및 서비스루틴이 정상적인 경우
    		if ( everythings_fine ) {
    			var msg = '결제가 완료되었습니다.';
    			msg += '\n고유ID : ' + rsp.imp_uid;
    			msg += '\n상점 거래ID : ' + rsp.merchant_uid;
    			msg += '\n결제 금액 : ' + rsp.paid_amount;
    			msg += '카드 승인번호 : ' + rsp.apply_num;

    			alert(msg);
    		} else {
    			//[3] 아직 제대로 결제가 되지 않았습니다.
    			//[4] 결제된 금액이 요청한 금액과 달라 결제를 자동취소처리하였습니다.
    		}
    	});
    } else {
        var msg = '결제에 실패하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

### naverProducts 파라미터<a id="naverProducts"></a>
 
`naverProducts`는 다음 6개의 속성으로 하나의 상품 정보를 표현하는 객체의 배열입니다. 

- categoryType (필수) : [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product) 참고
- categoryId (필수) : [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product) 참고
- uid (필수) : 가맹점 내부의 상품 고유 ID를 활용하는 것이 일반적이지만, 네이버페이 가이드 참고가 필요합니다. [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product)
- name (필수) : 주문상품의 명칭
- count (필수) : 상품 주문 개수
- sellerId (선택) : 가맹점이 하위 판매자를 식별하기 위한 고유 ID(영문 대소문자 및 숫자 허용)
	- 가맹점의 업종이 오픈마켓인 경우 필수 입력 
- payReferrer (선택) : 네이버 플랫폼의 타 서비스와 제휴계약 후 유입분석을 진행하는 경우에만 입력 [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product_ref)

### PC, 모바일 버전에서 진행 방식<a id="method"></a>

**PC 버전**
- Chrome80이슈(SameSite=Lax기본값 적용)에 대응하기 위해 결제예약 API를 활용한 팝업창 또는 리디렉션 방식으로 결제가 진행됩니다. 
- 네이버페이 공식 매뉴얼에는 Internet Explorer에서 window.opener 객체 유실 문제로 팝업창 방식을 제한한다고 안내되어 있으나, 아임포트는 팝업창 방식을 IE에서도 지원합니다.  

**모바일 버전**
- 네이버페이 신규 SDK를 활용해 리디렉션 또는 팝업창 방식으로 결제가 진행됩니다.
- 모바일 환경에서는 팝업창을 통한 결제에 다음과 같이 제약이 많기 때문에 페이지 리디렉션 방식으로 결제를 진행하는 것이 일반적이며 네이버페이 검수팀의 권장 사항입니다. PC와 모바일 환경 모두 리디렉션 방식으로 설정하면 동일한 결제 후처리 로직을 사용할 수 있는 장점이 있습니다.  
	- WebView를 통한 결제 : `window.open` 이벤트를 감지하여 별도의 WebView를 생성해야하는 Native 단 작업의 난이도 상승
	- Safari 등 브라우저 : 사용자 설정에 따라 강제로 팝업차단을 해버리는 경우 발생

## 3. 기타  

### 환불 API 요청 시 추가 속성 

아임포트 환불 API인 `POST /payments/cancel` 호출 시 다음 추가 속성를 설정해야 합니다.

- `extra.requester` : API를 호출하는 출처. 다음 중 선택 :
	- customer : 구매자에 의한 요청
	- admin(기본값) : 어드민에 의한 요청
- `reason`: 결제 취소 사유.

**예시(json)**
```javascript
{
  "imp_uid" : "imp_123412341234", //환불처리할 아임포트 거래번호
  "amount" : 3000, //환불할 금액
  "reason": "결제 취소 사유", //실제 사유와 같아야 함
  "extra" : {
    "requester" : "customer"
  }
}
```

**예시(form-urlencoded)**
```
imp_uid=imp_123412341234&amount=3000&extra[requester]=customer
```

## 4. 연동 주의사항

### 에러  메시지 비 가공
결제창 호출(IMP.request_pay 함수)후 결제창 하단의 “취소" 버튼 클릭 등으로 결제 프로세스가 중단되거나 잔액 부족, 한도 초과, 100원 미만 결제 등의 사유로 결제에 실패하면 콜백 함수(popup 방식)/m_redirect_url(리디렉션 방식)로 전달되는 결제 결과(response 객체/쿼리 파라메터)에 실패 사유(error_msg)가 전달됩니다. 이 에러 메시지는 사용자에게 가공 없이 그대로 노출되어야 합니다.
해당 규칙 미 준수시 네이버페이 실검수 진행시 수정요청 받게 됩니다.

예) error_msg가 “잔액 부족"이라고 가정할때, "결제에 실패하였습니다. 실패 사유:" + "잔액 부족"과 같은 형태로 가공되면 안됨

### 100원 미만 결제 처리

네이버페이 - 결제형의 최소 결제 금액은 100원이기 때문에, 100원 미만 결제 요청에 대해 예외 처리가 되어있어야 합니다.

예) 사용자에게 최소 결제 금액이 100원이라 결제를 할 수 없다는 의미를 담는 에러 메시지가 노출되어야 함


