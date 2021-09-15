일반 PG처럼 동작하는 (결제형) 네이버페이 방식에 대한 설명 문서입니다.

# 1. PC, 모바일 결제 공통  
PC버전의 경우 Chrome80이슈(SameSite=Lax기본값 적용)에 대응하기 위해 결제예약API를 활용한 팝업창 또는 리디렉션 방식으로 결제진행됩니다.
모바일 버전의 경우 네이버페이 신규 SDK를 활용해 리디렉션 또는 팝업창 방식으로 결제진행됩니다.

# 1.1 결제창 호출 javascript  

```javascript
IMP.request_pay({
    pg : 'naverpay',
    merchant_uid: "order_monthly_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    tax_free : 4000, //면세공급가액(누락되면 0원으로 처리)
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //누락되면 이니시스 결제창에서 오류
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverPopupMode : true, //리디렉션모드 vs 팝업모드
    naverProducts : [{ //상품정보(필수전달사항) 네이버페이 매뉴얼의 productItems 파라메터와 동일합니다.
      "categoryType": "BOOK",
			"categoryId": "GENERAL",
			"uid": "107922211",
			"name": "한국사",
			"payReferrer": "NAVER_BOOK",
			"count": 10
		},
		{
			"categoryType": "MUSIC",
			"categoryId": "CD",
			"uid": "299911002",
			"name": "러블리즈",
			"payReferrer": "NAVER_BOOK",
			"count": 1
		}]
}, function(rsp) {
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

# 1.2 PC결제  
PC 결제시,

- 팝업창을 통한 결제
- 페이지 리디렉션을 통한 결제

방식 중 선택하실 수 있습니다.

# 1.2.1 팝업창 방식  

`naverPopupMode : true` 옵션이 지정되면 팝업창을 통해 네이버페이 결제가 진행되며 결제성공/실패 후 `아임포트 콜백함수`가 실행됩니다.
네이버페이 공식 매뉴얼에서는 IE 결제시 window.opener 유실 문제가 설명되어있지만, 아임포트에서 다른 방식으로 해결하고 있으므로 IE에서도 동일하게 사용가능합니다.  

브라우저의 팝업차단기에 걸리지 않으려면 `IMP.request_pay()` 함수의 호출은 `onClick` 핸들러와 같이 사용자 context 에서 이루어져야 합니다.

```javascript
IMP.request_pay({
	//다른 결제 파라메터는 생략
	naverPopupMode : true
}, function(rsp) {
	//모든 브라우저에서 아임포트 콜백함수 실행
	if ( rsp.success ) {
		//결제 성공
	} else {
		//결제 실패
	}
});
```

# 1.2.2 페이지 리디렉션 방식  
모바일 환경에서는 팝업창을 통한 결제에 제약이 많습니다.   

- WebView를 통한 결제 : `window.open` 이벤트를 감지하여 별도의 WebView를 생성해야하는 Native 단 작업의 난이도 상승
- Safari 등 브라우저 : 사용자 설정에 따라 강제로 팝업차단을 해버리는 경우 발생

때문에, 모바일 환경에서는 페이지 리디렉션 방식으로 결제진행하는 것이 일반적입니다.(네이버페이 검수팀에서도 권장)
PC결제의 후처리 로직을 모바일과 동일하게 맞추길 원하시는 경우 PC결제 또한 페이지 리디렉션 방식으로 진행할 수 있습니다.

```javascript
IMP.request_pay({
	//다른 결제 파라메터는 생략
	naverPopupMode : false,
	m_redirect_url : "http://yourservice.com/payments/complete" //결제 후 URL로 리디렉션 되며 Query String 으로 imp_uid, merchant_uid 가 전달됨
});
```


# 1.3 모바일 결제  
모바일 결제시,

- 팝업창을 통한 결제
- 페이지 리디렉션을 통한 결제

방식 중 선택하실 수 있습니다.

# 1.3.1 팝업창 방식  

`naverPopupMode : true` 옵션이 지정되면 팝업창을 통해 네이버페이 결제가 진행되며 결제성공/실패 후 `아임포트 콜백함수`가 실행됩니다.

```javascript
IMP.request_pay({
	//다른 결제 파라메터는 생략
	naverPopupMode : true
}, function(rsp) {
	//모바일 결제 후 아임포트 콜백함수 실행
	if ( rsp.success ) {
		//결제 성공
	} else {
		//결제 실패
	}
});
```

# 1.3.2 페이지 리디렉션 방식  

```javascript
IMP.request_pay({
	//다른 결제 파라메터는 생략
	naverPopupMode : false, //기본값이 false 이므로 생략가능
	m_redirect_url : "https://yourservice.com/payments/complete" //결제 후 URL로 리디렉션 되며 Query String 으로 imp_uid, merchant_uid 가 전달됨
});
```

# 2. naverProducts 파라메터  

각 상품의 정보를 객체로 표현하며, 주문상품 종류의 개수만큼 배열로 전달되어야 합니다.  

하나의 상품정보는 아래 6가지 속성으로 표현됩니다.

- categoryType (필수) : [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product) 참고
- categoryId (필수) : [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product) 참고
- uid (필수) : 가맹점 내부의 상품 고유 ID를 활용하는 것이 일반적입니다만, 네이버페이 가이드 참고가 필요합니다. [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product)
- name (필수) : 주문상품의 명칭
- count (필수) : 상품 주문 개수
- payReferrer (선택) : 네이버 플랫폼의 타 서비스와 제휴계약 후 유입분석을 진행하는 경우에만 입력 [공식 매뉴얼](https://developer.pay.naver.com/docs/v2/api#etc-etc_product_ref)


# 3. 기타  

## 3.1 `name` 속성 주의사항  

`IMP.request_pay()` 호출 시, 전달되는 `name` 속성의 값으로 `xxxx 외 2개` 와 같은 표현은 사용하지 않습니다.  
`naverProduts` 파라메터를 분석하여 네이버페이 내부적으로 `외 2개` 의 표현이 자동생성되므로 상품명만 전달하시길 권장드립니다.

가장 일반적으로, `naverProducts[0].name` 과 같은 값을 전달하도록 개발하시길 권장드립니다.  

## 3.2 환불 API 요청 시 requester 속성 추가  

네이버페이의 경우, 구매자에 의해 환불 API가 요청되었는지 가맹점의 어드민에 의해 환불 API가 요청되었는지 구분하고 있습니다.
때문에, 아임포트 환불 API인 POST /payments/cancel 에 추가 필드가 정의되어있습니다.

환불 요청 시, `extra.requester` 필드가 추가되면 되며 다음과 같은 값을 전달주시면 됩니다.  
또한 reason 파라미터에 취소 사유를 입력하셔야 합니다.

- customer : 구매자에 의한 요청
- admin(기본값) : 어드민에 의한 요청

**예시(json)**
```javascript
{
  "imp_uid" : "imp_123412341234", //환불처리할 아임포트 거래번호
  "amount" : 3000, //환불할 금액
  "reason": 결제 취소 사유, 실제 사유와 같아야 함
  "extra" : {
    "requester" : "customer"
  }
}
```

**예시(form-urlencoded)**
```
imp_uid=imp_123412341234&amount=3000&extra[requester]=customer
```

## 3.3 이용완료일 처리  

결제하는 상품의 유형에 따라, 결제요청 시 이용완료일을 반드시 지정하도록 네이버페이-가맹점 간 계약되는 경우가 있습니다.  
결제창 호출을 위한 아임포트 javascript 호출 시, `naverUseCfm` 라는 파라메터를 추가해주셔야 합니다.(yyyyMMdd 형식의 문자열)  

- 형식 : yyyyMMdd 형식의 문자열. 결제 당일 또는 미래의 일자여야 함

**예시(javascript)**
```javascript
IMP.request_pay({
    pg : 'naverpay',
    merchant_uid: "order_monthly_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    tax_free : 4000, //면세공급가액(누락되면 0원으로 처리)
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //누락되면 이니시스 결제창에서 오류
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverUseCfm : '20201001', //이용완료일자 지정
    naverPopupMode : true, //리디렉션모드 vs 팝업모드
    naverProducts : [{ //상품정보(필수전달사항) 네이버페이 매뉴얼의 productItems 파라메터와 동일합니다.
      "categoryType": "BOOK",
            "categoryId": "GENERAL",
            "uid": "107922211",
            "name": "한국사",
            "payReferrer": "NAVER_BOOK",
            "count": 10
        },
        {
            "categoryType": "MUSIC",
            "categoryId": "CD",
            "uid": "299911002",
            "name": "러블리즈",
            "payReferrer": "NAVER_BOOK",
            "count": 1
        }]
}
```
