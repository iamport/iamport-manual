결제형 네이버페이를 활용한 반복결제 연동 가이드 문서입니다.   

# 1. PC, 모바일 결제 공통  
PC버전의 경우 Chrome80이슈(SameSite=Lax기본값 적용)에 대응하기 위해 결제예약API를 활용한 팝업창 또는 리디렉션 방식으로 결제진행됩니다.
모바일 버전의 경우 네이버페이 신규 SDK를 활용해 리디렉션 또는 팝업창 방식으로 결제진행됩니다.

## 1.1 결제고객 등록을 위한 javascript  

아임포트 SDK에 정의된 javascript 함수를 통해 결제고객 등록을 위한 네이버페이 결제화면을 호출할 수 있습니다.  

### `amount` 파라메터의 역할   
결제고객 등록과정에서는 결제승인이 이뤄지지 않습니다.   
`amount` 파라메터는 이후 API를 통한 반복결제 시 승인될 것으로 예상되는 금액을 결제창에 미리 표시하여 고객에게 정보를 제공하는 역할을 합니다.  

### `customer_uid` 파라메터  
`customer_uid` 파라메터의 유/무는 네이버페이 반복결제로 동작할지 일반결제로 동작할지를 결정하는 역할을 합니다.  
`customer_uid` 가 없으면 일반결제로 동작하며, 지정되면 반복결제 등록방식으로 동작합니다.  [일반결제 연동 가이드](https://github.com/iamport/iamport-manual/blob/master/NAVERPAY/sample/naverpay-pg.md)


반복결제를 위한 결제고객 등록이므로 `customer_uid`를 지정해주셔야 하고, 고객을 특정할 수 있는 가맹점 고유 KEY를 생성(80자 제한)해 전달주시면 됩니다.  

### `naverProductCode` 파라메터  
한 명의 고객이 판매되는 동일상품에 여러 번 반복결제 등록하는 것을 방지하려면 `naverProductCode` 파라메터를 **가맹점의 상품코드** 로 특정해주세요.  
기본값은 아임포트 내에서 Random 생성되므로 한 명의 고객이 제한없이 반복결제를 등록할 수 있습니다.   

```javascript
IMP.request_pay({
    pg : 'naverpay',
    customer_uid : '가맹점의 결제 고객을 특정하는 Unique Key', //customer_uid 를 활용해 실제 반복결제(승인요청)를 진행하게 됩니다.
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : 'Slim 요금제(1개월 단위)',
    amount : 14000, //실제 결제되지는 않으며, 향후 해당 금액으로 반복결제될 예정이라는 의미로 결제창에 표시될 금액만 전달합니다.
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //누락되면 이니시스 결제창에서 오류
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverProductCode : '반복결제 상품코드', //한 명의 고객이 동일 상품을 중복 결제할 수 없도록 하려면 naverProductCode를 지정해주세요. (누락되면 랜덤값으로 자동 생성)
    naverPopupMode : true //리디렉션모드 vs 팝업모드
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
    		//[2] 서버에서 REST API로 정상적으로 결제고객 등록이 이뤄졌는지 체크(status : paid 이면 등록 성공을 의미)
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

# 1.2 PC환경   

- 팝업창을 통한 결제
- 페이지 리디렉션을 통한 결제

방식 중 선택하실 수 있습니다.

# 1.2.1 팝업창 방식  

`naverPopupMode : true` 옵션이 지정되면 팝업창을 통해 네이버페이 결제고객 등록이 진행되며 등록성공/실패 후 `아임포트 콜백함수`가 실행됩니다.
네이버페이 공식 매뉴얼에서는 IE 결제시 window.opener 유실 문제가 설명되어있지만, 아임포트에서 다른 방식으로 해결하고 있으므로 IE에서도 동일하게 사용가능합니다.  

브라우저의 팝업차단기에 걸리지 않으려면 `IMP.request_pay()` 함수의 호출은 `onClick` 핸들러와 같이 사용자 context 에서 이루어져야 합니다.

```javascript
IMP.request_pay({
	//다른 결제 파라메터는 생략
	naverPopupMode : true
}, function(rsp) {
	//모든 브라우저에서 아임포트 콜백함수 실행
	if ( rsp.success ) {
		//결제고객 등록 성공
	} else {
		//결제고객 등록 실패
	}
});
```

# 1.2.2 페이지 리디렉션 방식  
모바일 환경에서는 팝업창을 통한 진행에 제약이 많습니다.   

- WebView를 통한 결제 : `window.open` 이벤트를 감지하여 별도의 WebView를 생성해야하는 Native 단 작업의 난이도 상승
- Safari 등 브라우저 : 사용자 설정에 따라 강제로 팝업차단을 해버리는 경우 발생

때문에, 모바일 환경에서는 페이지 리디렉션 방식으로 결제진행하는 것이 일반적입니다.(네이버페이 검수팀에서도 권장)
PC결제의 후처리 로직을 모바일과 동일하게 맞추길 원하시는 경우 PC결제 또한 페이지 리디렉션 방식으로 진행할 수 있습니다.

```javascript
IMP.request_pay({
	//다른 결제 파라메터는 생략
	naverPopupMode : false,
	m_redirect_url : "http://yourservice.com/payments/complete" //결제고객 등록 후 URL로 리디렉션 되며 Query String 으로 imp_uid, merchant_uid 가 전달됨
});
```


# 1.3 모바일 환경  

- 팝업창을 통한 결제
- 페이지 리디렉션을 통한 결제

방식 중 선택하실 수 있습니다.

# 1.3.1 팝업창 방식  

`naverPopupMode : true` 옵션이 지정되면 팝업창을 통해 네이버페이 결제가 진행되며 등록성공/실패 후 `아임포트 콜백함수`가 실행됩니다.

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
	m_redirect_url : "https://yourservice.com/payments/complete" //결제고객 등록 후 URL로 리디렉션 되며 Query String 으로 imp_uid, merchant_uid 가 전달됨
});
```

# 2. 최초 과금 및 이후 결제 승인 처리   

javascript 를 통해 결제고객 등록이 완료되더라도 승인처리는 이뤄지지 않으므로, 최초 과금을 위해서는 등록에 사용된 `customer_uid`를 사용해 직접 승인 API 호출이 필요합니다.  
또한, 이후 과금이 필요한 시점에 등록된 고객에 대한 결제를 위해서는 `customer_uid`를 사용해 승인 API를 호출해주셔야 합니다.  

아임포트에서 제공되는 REST API 중, [/subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again) 를 호출해 승인처리하실 수 있습니다. 

## merchant_uid 사용 주의사항  
반복결제 시 실수로 인해 중복과금되는 것을 방지하기 위해 승인처리에 한 번 사용된 `merchant_uid` 는 재사용이 불가능하도록 막혀있습니다.   
결제고객 등록 시 사용된 `merchant_uid` 또한 재사용이 불가능하므로 최초과금 또는 이후 과금을 위해 API 요청하실 때에는 서로 다른 `merchant_uid` 를 사용해주셔야 합니다.  

- customer_uid : 결제고객 등록 시 사용된 해당 고객의 customer_uid
- merchant_uid : 과금승인을 위한 가맹점 주문번호
- amount : 과금승인요청금액 (결제고객 등록 시 지정된 금액과 달라도 무방함)
- tax_free : amount 중 면세공급가액 (기본값은 0)
- name : 과금승인을 위한 주문의 명칭

**예시(json)**
```javascript
{
  "customer_uid" : "가맹점의 결제 고객을 특정하는 Unique Key", //결제고객 등록 시 사용된 customer_uid 와 같은 값
  "merchant_uid" : "가맹점 주문번호",
  "amount" : 10000,
  "name" : "Slim 요금제(최초과금)"
}
```

**예시(form-urlencoded)**
```
customer_uid={가맹점의 결제 고객을 특정하는 Unique Key}&merchant_uid={가맹점 주문번호}&amount=10000&name=Slim 요금제(최초과금)
```


# 3. 이용완료일 처리  

결제하는 상품의 유형에 따라, 결제요청 시 이용완료일을 반드시 지정하도록 네이버페이-가맹점 간 계약되는 경우가 있습니다.  
승인을 위한 아임포트 REST API 호출 시, `naverUseCfm` 라는 파라메터를 `extra` 필드로 전송해주셔야 합니다.(yyyyMMdd 형식의 문자열)  

- 승인 요청 API : [/subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)
- 일반 상품 대비 추가되는 파라메터 : extra.naverUseCfm
- 형식 : yyyyMMdd 형식의 문자열. 결제 당일 또는 미래의 일자여야 함

**예시(json)**
```javascript
{
  "customer_uid" : "가맹점의 결제 고객을 특정하는 Unique Key", //결제고객 등록 시 사용된 customer_uid 와 같은 값
  "merchant_uid" : "가맹점 주문번호",
  "amount" : 10000,
  "name" : "Slim 요금제(최초과금)",
  "extra" : {
    "naverUseCfm" : "20201001"
  }
}
```

**예시(form-urlencoded)**
```
customer_uid={가맹점의 결제 고객을 특정하는 Unique Key}&merchant_uid={가맹점 주문번호}&amount=10000&name=Slim 요금제(최초과금)&extra[naverUseCfm]=20201001
```