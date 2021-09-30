# 네이버페이 배송비 정책 및 예시

본 문서는 네이버페이 결제 시 적용 가능한 배송비 정책을 다음 유형별로 설명합니다.  

- [1. 무료 배송](#free)
- [2. 고정 배송비](#fixed)
- [3. 조건부 무료 배송](#conditional)
- [4. 수량별 차등 배송비](#quantity-based)
- [5. 지역별 추가 배송비](#location-based)
  
## `naverProducts.shipping` 속성  

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay) 호출 시 `naverProducts.shipping` 속성에 대한 JSON 구성은 다음과 같습니다.  

```javascript
{
	groupId : "묶음배송구분 카테고리 ID",
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 400, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : {
		freeByThreshold : 20000, //조건부 무료 배송 임계 값
		repeatByQty : 4, //수량별 차등 부과(규칙적인 추가 배송비)
		rangesByQty : [ //수량별 차등 부과(수량 구간별 커스텀 추가 배송비)
			{from: 10, surcharge:4000},
			{from: 20, surcharge:6000}
		],
		surchargesByArea : [ //지역별 추가 배송비
			{area:"island", surcharge:2000}, //일반 도서산간 지역
			{area:"jeju", surcharge:3000} // 제주도
		]
		//	surchargesByArea : "API" // API를 호출하여 추가배송비 결정 (이용 시 지역별 추가 배송비 설정은 삭제)
	}
}
```

## 1. 무료 배송<a id="free"/>

무료 배송은 `feeRule`, `feePayType` 속성을 정의할 필요가 없으며 다음과 같이 설정합니다.  

```javascript
{
	//* ...생략... *//
	baseFee : 0 //기본 배송비
}
```

## 2. 고정 배송비<a id="fixed"/>  

고정 배송비를 부과하려면 `feeRule` 속성을 정의할 필요가 없으며 다음과 같이 설정합니다.  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED" //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
}
```

## 3. 조건부 무료 배송<a id="conditional"/>

기본적으로 배송비가 부과되나 주문 총합계 금액이 기준 이상일 때 배송비를 면제해 줄 수 있습니다.  

다음과 같이 설정하면 기본 배송비가 2,500원이며 주문 총합계 금액이 20,000원 이상일 때는 배송비가 무료입니다.  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : { //배송비 정책
		freeByThreshold : 20000 //무료 배송 임계 값
	}
}
```

## 4. 수량별 차등 배송비<a id="quantity-based"/>  

부피가 크거나 중량이 무거운 상품을 여러개 구매할 경우 추가 배송비를 부과할 수 있습니다.  

### 4.1 규칙적인 추가 배송비

상품 구매수량이 N개 늘어날 때마다 기본배송비 기준으로 배송비가 일정하게 증가하는 방법입니다.  

다음과 같이 설정하면 기본 배송비가 2,500원이며 10개 마다 2,500원의 배송비가 추가됩니다.  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : { //배송비 정책
		repeatByQty : 10 //추가배송비가 과금되는 구매수량
	}
}
```

### 4.2 수량 구간별 커스텀 추가 배송비  

수량 구간을 최대 3개로 나누어 기본배송비 기준으로 각 구간별로 배송비를 추가하는 방법입니다.  

아래와 같이 설정하면 다음의 배송비가 부과됩니다.

- 1개~10개까지 : 기본배송비 = 총 2500원
- 11개~20개까지 : 기본배송비 + 2,5000 = 총 5,000원
- 21개 이상: 기본배송비 + 7,5000 = 총 10,000원  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : { // 배송비 정책
		rangesByQty : [ // 수량별 구간 정의
			{from: 11, surcharge:2500}, //11개이상 21개 미만 2500원 배송비 추가
			{from: 21, surcharge:7500} //21개이상 7,500원 배송비 추가
		]
	}
}
```


## 5. 지역별 추가 배송비<a id="location-based"/>

상품 배송지역에 따른 추가 배송비를 설정할 수 있습니다.  

### 5.1 우편번호 기준으로 네이버페이에 위임  

네이버페이에서 배송지역의 우편번호을 기준으로 최대 3개의 구간으로 추가배송비를 자동 계산하도록 설정할 수 있습니다.  

1. 기본 내륙 배송비 : `baseFee` 적용
2. 도서 산간 지역(거제도, 울릉도, 강화도 등) : *area명은 island입니다.*
3. 제주도 : *area명은 jeju입니다.* 구간 정의가 없으면 제주도는 도서 산간 지역으로 간주됩니다.  

#### 도서산간 지역(제주도 포함) 추가 배송비  

다음과 같이 설정하면 기본 내륙 배송비가 2,500원이며 제주도를 포함한 모든 도서산간 지역은 2,000원이 추가되어 총 4,500원의 배송비가 부과됩니다.  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : {
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000} // 제주도 포함 도서산간 지역 추가배송비
		]
	}
}
```

#### 도서산간 지역(제주도 별도) 추가 배송비  

아래과 같이 설정하면 기본 내륙 배송비가 2,500원이며 다음과 같이 배송비가 부과됩니다.

- 일반 도서산간 지역 : 기본배송비 + 2,000 = 총 4,500원  
- 제주도 : 기본배송비 + 4,500 = 총 7,000원  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : {
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000}, //일반 도서산간 추가 배송비
			{area:"jeju", surcharge:4500} //제주도 추가 배송비
		]
	}
}
```

### 5.2 커스텀 지역 배송비  

꽃배달, 퀵서비스와 같이 시,군,구 단위로 지역배송비 설정을 보다 세밀하게 해야하는 경우 API를 이용해 커스텀 구현할 수 있습니다.  

다음과 같이 설정하면 네이버페이는 API를 호출하여 지역배송비를 가맹점에 요청합니다. 이를 위해 가맹점은 우편번호 DB를 구축하여야 합니다.  

```javascript
{
	//* ...생략... *//
	baseFee : 2500, //기본 배송비
	feePayType : "PREPAYED", //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
	feeRule : {
		surchargesByArea : "API" // API를 호출하여 추가배송비 결정
	}
}
```

#### 지역 배송비 요청과 응답   

네이버페이는 주문 페이지에서 네이버페이 가입 시 사전등록된 `도서산간비 정보요청 URL`로 입력된 배송 주소에 대한 HTTP 요청을 보내 지역배송비 정보를 XML 형식으로 받습니다.  

**HTTP 요청 예시**

- productId[순번] : 배송비 조회하려는 상품의 상품번호
- zipcode : 배송지 우편번호
- address1 : 배송지 기본 주소(base64 url encoding된 값)

```
http://{도서산간비 정보요청 URL}?productId[0]=XXX&productId[1]=XXX&zipcode=463050&address1=6rK96riw64-EIOyEseuCqOyLnCDrtoTri7nqtawg7ISc7ZiE64-Z
```

**XML 응답 예시 (Content-Type: application/xml)**

```xml
<additionalFees> <!-- query 파라미터로 전달된 각 productId별로 노드 구성 -->
   <additionalFee> <!-- 상품별 추가배송비 정의  -->
          <id>P001</id> <!-- productId(상품번호) -->
          <surprice>0</surprice> <!-- 추가 배송비 -->
   </additionalFee>
   <additionalFee>
          <id>P002</id>
          <surprice>1000</surprice>
   </additionalFee>
<additionalFees>
```

