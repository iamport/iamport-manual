네이버페이 결제 시 적용 가능한 배송비 정책을 유형별로 알아보고, 파라메터 구성에 대한 예시를 확인해봅니다.  

# JSON 구성  

`IMP.request_pay(param, callback)` 호출 시 `naverProducts.shipping` 속성에 대한 JSON구성은 다음과 같습니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 400, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		freeByThreshold : 20000,
		repeatByQty : 4,
		rangesByQty : [
			{from: 10, surcharge:4000},
			{from: 20, surcharge:6000}
		],
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000},
			{area:"jeju", surcharge:3000}
		]
	}
}
```

# 1. 무료 배송  

`feeRule`, `feePayType` 속성을 정의할 필요가 없으며 

- baseFee : 0

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 0 //기본 배송비
}
```

# 2. 배송비 고정  
모든 구매자에게 동일한 배송비를 수취하는 경우에 사용합니다.(예. 2,500원)  

`feeRule` 속성을 정의할 필요가 없으며 

- baseFee : 2500
- feePayType : "PREPAYED"(선불) 또는 "CASH\_ON\_DELIVERY"(착불)

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED" //PREPAYED(선불), CASH_ON_DELIVERY(착불)
}
```

# 3. 조건부 무료 배송(feeRule)  
기본적으로 배송비가 부과되나 주문 총합계 금액이 기준 이상일 때 배송비를 면제해주는 정책인 경우입니다.  

기본 배송비는 2,500원이며 주문 총합계 금액이 20,000원 이상일 때 배송비 무료를 설정하기 위해서는 

- feeRule.freeByThreshold : 20000
- baseFee : 2500
- feePayType : "PREPAYED"(선불) 또는 "CASH\_ON\_DELIVERY"(착불)

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		freeByThreshold : 20000
	}
}
```

# 4. 구매 수량별 차등 배송비(feeRule)  
제품의 부피가 크거나 중량이 무거워 구매자가 상품을 여러 개 구매할 경우 배송비가 추가로 부과되어야하는 경우입니다.  

## 4.1 규칙적인 배송비 부과  
상품 구매수량이 N개 늘어날 때마다 배송비가 일정하게 증가하는 경우입니다.  
(기본배송비 기준으로 금액이 증가하게 됩니다.)

기본 배송비는 2,500원이며 10개마다 2,500원의 배송비가 반복적으로 증가하도록 설정하려면, 

- feeRule.repeatByQty : 10
- baseFee : 2500
- feePayType : "PREPAYED"(선불) 또는 "CASH\_ON\_DELIVERY"(착불)

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		rangesByQty : 10
	}
}
```

## 4.2 수량 구간별 커스텀 추가 배송비(feeRule)  
수량 구간을 임의로 나누어 각 구간별로 **추가 부과**되는 배송비를 설정할 수 있습니다. 구간은 최대 3개까지만 설정이 가능합니다.  

구매 수량이 1개~10개까지는 기본배송비를 적용하며 11개~20개까지는 5,000원의 배송비(== 2,500원 추가)를, 21개 이상은 10,000원(== 7,500원 추가)의 배송비를 부과하려면 

- feeRule.rangesByQty : 구간정의
- baseFee : 2500
- feePayType : "PREPAYED"(선불) 또는 "CASH\_ON\_DELIVERY"(착불)

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		rangesByQty : [
			{from: 11, surcharge:2500}, //11개이상 21개 미만 최종 5,000(=2,500+2,500)원 배송비
			{from: 21, surcharge:7500} //21개이상 최종 10,000(=2,500+7,500)원 배송비
		]
	}
}
```


# 5. 지역별 추가배송비(surchargesByArea)   
상품 배송지역에 따른 추가 배송비를 결정하는 설정이므로, #1, #2, #3, #4에서 안내된 feeRule과 동시에 선언될 수 있습니다.  

(즉, feeRule에서 `surchargesByArea`는 `freeByThreshold`, `repeatByQty`, `rangesByQty`와 함께 선언될 수 있습니다). 

## 5.1 우편번호 기준으로 네이버페이에 위임  
네이버페이에서 배송지역의 우편번호를 보고 추가배송비를 자동 계산하도록 설정하는 방법입니다. 최대 3개의 구간으로 구분될 수 있으며, 구간의 정의는 다음과 같습니다.  

1. 기본 내륙 배송비
2. 도서 산간 지역(거제도, 울릉도, 강화도 등) : *area명은 island입니다.*
3. 제주도 : *area명은 jeju입니다.* 

기본 내륙 배송비는 `baseFee`금액이 적용됩니다.  
제주도에 대한 구간 정의가 이뤄지지 않으면, 제주도는 일반 도서 산간지역으로 분류됩니다.  

### 5.1.1 제주포함, 도서산간 지역 배송비 추가  

기본 내륙 배송비는 2,500원으로 설정하고 제주도를 포함하여 모든 도서산간 지역은 4,500원의 배송비(== 2,000원 추가 배송비)를 책정하고자 한다면, 

- feeRule.surchargesByArea : `{area:"island", surcharge:2000}`
- baseFee : 2500
- feePayType : "PREPAYED"(선불) 또는 "CASH\_ON\_DELIVERY"(착불)

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000}
		]
	}
}
```


### 5.1.2 일반 도서산간 지역과 제주도 구분한 배송비 설정  

기본 내륙 배송비는 2,500원으로 설정하고 일반 도서산간 지역은 4,500원(== 2,000원 추가 배송비)으로, 제주도의 경우 7,000원(== 4,500원 추가 배송비)의 배송비를 책정하고자 한다면,  

- feeRule.surchargesByArea : `{area:"island", surcharge:2000}, {area:"jeju", surcharge:4500}`
- baseFee : 2500
- feePayType : "PREPAYED"(선불) 또는 "CASH\_ON\_DELIVERY"(착불)

로 설정합니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000},
			{area:"jeju", surcharge:4500}
		]
	}
}
```

## 5.2 지역배송비 커스텀 설정  

네이버페이 기본 기능에서는 내륙 / 일반 도서산간 / 제주도 3가지 구분만 가능하기 때문에 꽃배달, 퀵서비스와 같이 시,군,구 단위로 지역배송비 설정을 보다 세밀하게 해야하는 경우 API를 이용해 자체적으로 구현할 수 있습니다.  
이 경우에는 가맹점에서 자체적으로 우편번호 DB를 구축하고 구매자가 구매를 시도할 때 네이버페이 => 가맹점으로 요청하는 API에 대해 응답함으로써 지역에 대한 추가배송비를 설정할 수 있습니다.(추후 보다 상세한 내용 업데이트하겠습니다)  

- feeRule.surchargesByArea : "API"

와 같이 "API"라는 문자열(string)을 지정하면 됩니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불), CASH_ON_DELIVERY(착불)
	feeRule : {
		surchargesByArea : "API"
	}
}
```

### 5.2.1 지역배송비 정보 요청과 XML 응답   

네이버페이는 주문서 작성 페이지에서 네이버페이 가입시 사전등록된 `도서산간비 정보요청 URL`로 HTTP 요청을 보내 지역배송비 정보를 XML형태로 받아갑니다.(네이버페이 주문서작성 페이지에서 구매자가 입력하는 주소에 따라 HTTP요청이 이뤄집니다.)  

네이버페이로부터 전송되는 HTTP요청은 다음과 같은 형태를 띄고 있습니다. 

```
http://{도서산간비 정보요청 URL}?productId[0]=XXX&productId[1]=XXX&zipcode=463050&address1=6rK96riw64-EIOyEseuCqOyLnCDrtoTri7nqtawg7ISc7ZiE64-Z
```

요청시 전송되는 query string 파라메터는, 

- productId[순번] : 배송비 조회하려는 상품의 상품번호
- zipcode : 배송지 우편번호
- address1 : 배송지 기본 주소(base64 url encoding된 값)

위 요청에 대한 응답은 다음과 같은 형태의 XML이어야 합니다.(Content-Type: application/xml)

```xml
<additionalFees>
   <additionalFee>
          <id>P001</id>
          <surprice>0</surprice>
   </additionalFee>
   <additionalFee>
          <id>P002</id>
          <surprice>1000</surprice>
   </additionalFee>
<additionalFees>
```

- /additionalFees/additionalFee[순번] : query 파라메터로 전달된 각 productId별로 node구성
- /additionalFees/additionalFee[순번]/id : productId(상품번호)
- /additionalFees/additionalFee[순번]/surprice : 추가되는 배송비

