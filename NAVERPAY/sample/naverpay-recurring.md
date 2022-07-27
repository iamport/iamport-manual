# 결제형 네이버페이 정기/반복결제 연동 가이드

:globe_with_meridians: [EN](/en/NAVERPAY/sample/naverpay-recurring.md)  

결제형 네이버페이 정기/반복결제는 정기/반복결제 상품의 결제 수단으로 네이버페이를 선택하여 결제가 진행됩니다.

## 1. PG 설정하기

<a href="https://guide.iamport.kr/485c6da8-01d7-4900-bc05-76005e5477ba" target="_blank">네이버페이(주문형/결제형) 테스트 모드 설정</a> 페이지에서 **2) 결제형** 내용을 참고하여 PG 설정을 합니다.


## 2. 정기/반복결제 등록창 호출하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 정기/반복결제 등록창을 호출합니다.  

PC와 모바일 모두 `IMP.request_pay(param, callback)` 호출 후 파라미터(`naverPopupMode`) 설정을 통해 팝업 방식(callback 호출) 또는 리디렉션 방식(`m_redirect_url`로 페이지 이동)으로 진행될 수 있습니다. [PC, 모바일 버전에서 진행 방식](#method)을 참고하세요.  

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `naverpay`로 지정합니다.
- `customer_uid` : 고객을 특정할 수 있는 unique key. 
	- 정기/반복 결제 등록을 위해서 지정해야 합니다. 미 지정 시, 일반결제로 진행됩니다.
	- 등록 후 해당 키를 사용해 반복결제 승인을 요청할 수 있습니다.
- `amount` : 정기/반복 결제 등록과정에서는 결제승인이 이뤄지지 않습니다. (등록 후 결제할 금액을 고객에 표시하는 용도)
- `buyer_tel` : 필수 입력.
- `naverPopupMode` : 팝업 방식으로 진행 여부 (true/false).
	- `false`인 경우, 페이지 리디렉션 방식으로 진행되며 `m_redirect_url`을 설정해야 합니다.
- `m_redirect_url` : 리디렉션 방식으로 진행(`naverPopupMode`: false)할 경우 결제 완료 후 리디렉션 될 URL. 
- `naverProductCode` : 가맹점의 상품코드.
	- 동일한 고객이 동일상품에 대해 중복으로 반복결제 등록하는 것을 방지하기 위한 파라미터입니다.
	- 기본값은 random으로 자동 생성되어 중복결제가 가능하므로 값을 지정해 주세요.
- `naverChainId` : 네이버페이 그룹형 가맹점용 chain id
    - 같은 파트너 ID로 두개 이상의 서비스를 운영하는 그룹형 가맹점의 경우에만 네이버페이로부터 전달받은 값을 필수 입력합니다.
    - 비대상 가맹점은 입력하지 않습니다.
  
```javascript
IMP.request_pay({
    pg : 'naverpay',
    customer_uid : 'gildong_0001_1234', //고객 식별키, 필수 입력.
    merchant_uid: "order_monthly_0001", //상점에서 생성한 고유 주문번호
    name : 'Slim 요금제(1개월 단위)',
    amount : 14000, //등록 후 API로 결제 할 금액.
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //필수 입력.
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverPopupMode : true, //팝업모드 활성화
    m_redirect_url : "{등록 완료 후 리디렉션 될 URL}", //예 : http://yourservice.com/payments/complete
    naverProductCode : '반복결제 상품코드',
    naverChainId: "sAMplEChAINid",
}, function(rsp) { //팝업 방식으로 진행 또는 등록 프로세스 시작 전 오류가 발생할 경우 호출되는 callback
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
### PC, 모바일 버전에서 진행 방식<a id="method"></a>

**PC 버전**
- Chrome80이슈(SameSite=Lax기본값 적용)에 대응하기 위해 정기/반복결제 등록 API를 활용한 팝업창 또는 리디렉션 방식으로 등록이 진행됩니다. 
- 네이버페이 공식 매뉴얼에는 Internet Explorer에서 window.opener 객체 유실 문제로 팝업창 방식을 제한한다고 안내되어 있으나, 아임포트는 팝업창 방식을 IE에서도 지원합니다.  

**모바일 버전**
- 네이버페이 신규 SDK를 활용해 리디렉션 또는 팝업창 방식으로 등록이 진행됩니다.
- 모바일 환경에서는 팝업창을 통한 등록에 다음과 같이 제약이 많기 때문에 페이지 리디렉션 방식으로 등록를 진행하는 것이 일반적이며 네이버페이 검수팀의 권장 사항입니다. PC와 모바일 환경 모두 리디렉션 방식으로 설정하면 동일한 등록 후처리 로직을 사용할 수 있는 장점이 있습니다.  
	- WebView를 통한 등록 : `window.open` 이벤트를 감지하여 별도의 WebView를 생성해야하는 Native 단 작업의 난이도 상승
	- Safari 등 브라우저 : 사용자 설정에 따라 강제로 팝업차단을 해버리는 경우 발생


## 3. 최초 결제 및 이후 결제 예약하기   

정기/반복결제 등록이 완료되면 등록된 `customer_uid`를 사용해 결제승인 API를 호출하여 결제를 요청하거나 결제를 예약할 수 있습니다. 아임포트 REST API를 사용하여 정기/반복결제를 구현하는 방법은 [정기결제 연동하기](https://docs.iamport.kr/implementation/subscription)를 참고하세요.  

ℹ️ 정기/반복결제 시 실수로 인해 중복과금을 방지하기 위해 결제 등록 및 최초/반복 결제 승인을 위해 API 호출 시 매번 다른 `merchant_uid`(가맹점 주문번호)를 사용해야 합니다. 즉, 한번 사용한 `merchant_uid`는 재사용이 불가능합니다.  
 
## 3.1 최초/이후 결제 요청하기

REST API [/subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again) 를 호출해 결제 승인 요청을 합니다.

- `customer_uid` : 정기/반복결제 등록 시 사용된 해당 고객의 `customer_uid`
- `merchant_uid` : 가맹점 주문번호
- `amount` : 결제승인 요청금액 (결제고객 등록 시 지정된 금액과 달라도 무방함)
- `tax_free` : `amount` 중 면세공급가액 (기본값: 0)
- `name` : 주문의 명칭
- `extra.naverUseCfm` : 이용 완료일(yyyyMMdd 형식의 문자열로 결제 당일 또는 미래의 일자여야 함).
	-   상품 유형에 따라, 네이버페이-가맹점 간 필수값으로 계약되는 경우 입력합니다. 

**예시(json)**
```javascript
{
  "customer_uid" : "gildong_0001_1234",
  "merchant_uid" : "order_monthly_0002", //재사용 불가
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

## 3.2 결제 예약하기

REST API [/subscribe/payments/schedule](https://api.iamport.kr/#!/subscribe/schedule)를 호출해 결제 예약 요청을 합니다.

- `customer_uid` : 정기/반복결제 등록 시 사용된 해당 고객의 `customer_uid`
- `schedules` : 결제 예약 정보 객체 배열(1개 이상 설정 가능)
	- `merchant_uid` : 가맹점 주문번호
	- `schedule_at` : 결제요청 예약시각 (UNIX timestamp)
	- `amount` : 결제승인 요청금액 (결제고객 등록 시 지정된 금액과 달라도 무방함)
	- `extra.naverUseCfm` : 이용 완료일(yyyyMMdd 형식의 문자열로 결제 당일 또는 미래의 일자여야 함).
		- 상품 유형에 따라, 네이버페이-가맹점 간 필수값으로 계약되는 경우 입력합니다. 

**예시(json)**
```javascript
{
  "customer_uid" : "gildong_0001_1234",
  "schedules": [
    {
      "merchant_uid" : "order_monthly_0003", //재사용 불가
      "schedule_at" : 1519862400,
      "amount" : 10000,
      "extra" : {
        "naverUseCfm" : "20201001"
      }
    }
  ]
}
```

**예시(form-urlencoded)**
```
customer_uid={가맹점의 결제 고객을 특정하는 Unique Key}&schedules[0][merchant_uid]={가맹점 주문번호}&schedules[0][schedule_at]={결제요청 예약시각 UNIX timestamp}&schedules[0][amount]=10000&schedules[0][extra][naverUseCfm]=20201001
```
