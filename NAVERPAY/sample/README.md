현재 제공되는 네이버페이 결제는, 구매자가 주문하는 상품정보가 연동되어야하는 (주문형)네이버페이 방식입니다. 상품정보 연동없이 결제기능만 제공되는 (결제형)네이버페이 방식은 추후 별도 지원될 수 있습니다.(시기는 미정)  

# 1. PC, 모바일 연동 공통  

네이버페이 결제는 그 특성상 결제 완료 여부를 곧바로 전달받을 수 없어, 결제가 완료된 시점에 `IMP.request_pay(param, callback)`의 callback 함수가 호출되지 않습니다.  
(단, 네이버페이 결제프로세스가 시작되기 전까지 데이터 검증 등에 실패하는 경우에는 callback 함수가 호출됩니다.)  
같은 이유로 `m_redirect_url`파라메터도 동작하지 않습니다. 

결제완료여부 연동에 대한 내용은 [4. 결제정보 동기화](#4-결제정보-동기화) 섹션을 참고해주세요.  


- 네이버페이를 "기본PG사"로 설정해 하나만 사용하시는 경우에는 `pg`파라메터는 생략이 가능합니다. (관리자 페이지에 설정된 정보를 기본으로 사용합니다)  
- 복수개의 PG설정을 해두신 경우 네이버페이 결제창을 호출하시려면 `pg`파라메터를 `naverco`로 설정해주세요.  


```javascript
IMP.request_pay({
    pg : 'naverco',
    pay_method : 'card', //연동되지 않습니다. 네이버페이 결제창 내에서 결제수단을 구매자가 직접 선택하게 됩니다.
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //누락되면 이니시스 결제창에서 오류
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverProducts : [
    	{
    		id : "singleProductId",
    		name : "네이버페이 상품1",
    		basePrice : 1000,
    		taxType : 'TAX_FREE', //TAX or TAX_FREE
    		quantity : 2,
    		infoUrl : "http://www.iamport.kr/product/detail",
    		imageUrl : "http://www.iamport.kr/product/detail/thumbnail",
    		shipping : {
    			groupId : "shipping-a",
    			method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
    			baseFee : 2500,
    			feeRule : {
    				freeByThreshold : 20000
    			},
    			feePayType : "PREPAYED" //PREPAYED(선불), CASH_ON_DELIVERY(착불)
            },
            supplements : [
                {
                    id : "supplement-a",
                    name : "추가구성품 A",
                    price : 1000,
                    quantity : 1
                },
                {
                    id : "supplement-b",
                    name : "추가구성품 B",
                    price : 1200,
                    quantity : 2
                }
            ]
    	},
    	{
    		id : "optionProductId",
    		name : "네이버페이 상품2",
    		basePrice : 1000,
    		taxType : 'TAX_FREE', //TAX or TAX_FREE
    		infoUrl : "http://www.iamport.kr/product/detail",
    		imageUrl : "http://www.iamport.kr/product/detail/thumbnail",
    		options : [ //네이버페이 상품2에 대해서 빨강-170mm사이즈 3개와 빨강-180mm사이즈 2개: 총 5개 구매
    			{
    				optionQuantity : 3,
    				optionPrice : 100,
    				selectionCode : "R_M",
    				selections : [
    					{
	      					code : "RED",
	  						label : "색상",
	  						value : "빨강"
	  					},
	  					{
	  						code : "170",
	  						label : "사이즈",
	  						value : "170"
	  					}
					]
				},
    			{
    				optionQuantity : 2,
    				optionPrice : 200,
    				selectionCode : "R_L",
    				selections : [
    					{
	      					code : "RED",
	  						label : "색상",
	  						value : "빨강"
	  					},
	  					{
	  						code : "180",
	  						label : "사이즈",
	  						value : "180"
	  					}
					]
				}
			],
			shipping : {
				groupId : "shipping-a",
				method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
				baseFee : 2500,
				feeRule : {
    				freeByThreshold : 20000
    			},
				feePayType : "PREPAYED" //PREPAYED(선불), CASH_ON_DELIVERY(착불)
			}
		}
	]
});
```

## 1.1 네이버 디자인 가이드  
네이버페이 구매하기 기능은 네이버페이가 제공하는 SDK를 통해 지정된 디자인을 그대로 사용해 구현하셔야 합니다. 
![](screenshot/naver-button.png)

**네이버페이 버튼 UI생성을 위한 SDK**  

```html
<!-- PC용 -->
<script type="text/javascript" src="//pay.naver.com/customer/js/naverPayButton.js"></script>

<!-- 모바일용 -->
<script type="text/javascript" src="//pay.naver.com/customer/js/mobile/naverPayButton.js"></script>
```
위 스크립트 추가 후, 아래의 코드를 통해 지정된 HTML element에 네이버페이 버튼 UI가 생성되도록 호출할 수 있습니다. 
아래 코드는 HTML element를 네이버페이 버튼 UI로 변환할 뿐 자체적인 동작기능은 없습니다. 때문에 `NPay구매버튼 클릭`, `찜버튼 클릭`에 해당되는 핸들러를 직접 구현해주어야 합니다.  

핸들러는 각각 `BUY_BUTTON_HANDLER`, `WISHLIST_BUTTON_HANDLER` 속성으로 지정할 수 있으며, 핸들러 내에서 아임포트 javascript API를 호출하면 됩니다.  

```javascript
naver.NaverPayButton.apply({
	BUTTON_KEY: "네이버에서 전달받은 버튼생성키",
	TYPE: "C", //버튼 스타일
	COLOR: 1, //버튼 색상타입
	COUNT: 2, // 네이버페이버튼 + 찜하기버튼 모두 출력 여부
	ENABLE: "Y", //네이버페이 활성여부(재고부족 등에는 N으로 비활성처리)
	EMBED_ID: "iamport-naverpay-product-button", //네이버페이 버튼 UI가 부착될 HTML element의 ID
	BUY_BUTTON_HANDLER : function() {
		//중략
		//핸들러 내에서 아임포트 네이버페이 결제 함수 호출
		IMP.request_pay(param);
	},
	WISHLIST_BUTTON_HANDLER : function() {
		//중략
		
		//핸들러 내에서 아임포트 네이버페이 찜하기 함수 호출(iamport.payment-1.1.6.js부터 지원됨)
		IMP.naver_zzim(param);
	}
});
```



# 2. 상품정보 연동  
네이버페이 결제를 위해서는 구매할 상품정보 전달이 필수입니다. `IMP.request_pay(param, callback)` 호출 시 상품정보를 `param.naverProducts`파라메터로 전달해주셔야 합니다.  

추가로 네이버페이 구매에 대한 유입경로 분석을 하기 위해서는 `IMP.request_pay(param, callback)` 호출 시 `param.naverInterface`파라메터로 관련 정보를 전달해주셔야 합니다.  

## 2.1 param.naverProducts 의 구조  

Full JSON schema 구조 : [naverProducts JSON Schema](naverpay-schema.md)

`naverProducts` 는 Product의 array로 표현됩니다.  

### Product  

구매할 개별 상품에 대한 상세 정보와 함께 구매자가 해당 상품에 대해 선택한 옵션들(options) 및 적용될 배송비 정책(shipping)을 설정합니다.  

```javascript
{
	"id" : "Shoe_ax82a",   //상품고유ID
	"merchantProductId" : "Shoe_ax82a", //상품관리ID(필요한 경우만 선언. 정의하지 않으면 id값과 동일한 값을 자동 적용합니다)
	"ecMallProductId" : "Shoe_ax82a",   //지식쇼핑상품관리ID(필요한 경우만 선언. 정의하지 않으면 id값과 동일한 값을 자동 적용합니다)
	"name" : "신발", //상품명
	"basePrice" : 1000, //상품가격
	"taxType" : "TAX",       //부가세 부과 여부(TAX or TAX_FREE)
	"quantity" : 2,  //상품구매수량
	"infoUrl" : "http://www.iamport.kr/product/detail",    //상품상세페이지 URL
	"imageUrl" : "http://www.iamport.kr/product/detail/thumbnail",   //상품 Thumbnail 이미지 URL
	"options" : "array(of option)",     //구매자가 선택한 상품 옵션에 대한 상세 정보
	"shipping" : "object(of shipping)"    //상품 배송관련 상세 정보
}
```
Product는 `options` 과 `supplements`를 객체의 배열, `shipping` 을 객체로 관리합니다.  

### Option  
구매자가 **선택한 옵션**에 대한 상세 내용을 포함하고 있습니다.  

(아래 예시는, 신발(Product)상품에 대해 색상(파랑,빨강,초록) & 사이즈(160, 170, 180)이라는 상품옵션이 제공되고 있을 때, 구매자가 색상 옵션은 빨강을 선택하고 사이즈는 180 옵션을 선택한 경우를 나타내고 있습니다.)  

**색상**  

- 파랑
- 빨강(구매자가 선택함)
- 초록

**사이즈**

- 150mm
- 160mm
- 170mm
- 180mm(구매자가 선택함)
  
```javascript
{
	optionQuantity : 2,     //해당 옵션이 선택된 상품의 구매수량
	optionPrice : 200,      //옵션 선택에 따른 추가금액. Product.basePrice와 합산되며, 마이너스(-)기호를 포함할 수 있음
	selectionCode : "R_L",     //구매자가 선택한 옵션조합에 대한 관리코드. RED옵션과 180옵션을 선택했기 때문에 이를 의미하는 R_L코드를 정의(가맹점별로 직접 자유롭게 결정하면 됩니다)
	selections : [             //구매자가 선택한 옵션에 대한 상세 내용
		{
			code : "RED",       //해당 옵션의 구분 코드
			label : "색상",      //해당 옵션에 대한 라벨 명칭
			value : "빨강"       //해당 옵션의 값(코드에 대한 값의 표기명)
		},
		{
			code : "180",
			label : "사이즈",
			value : "180"
		}
	]
}
```

### Supplement  
상품과 연관된 추가구성품를 포함하여 구매하는 경우, 추가구성품에 대한 상세 정보를 포함하고 있습니다.  
하나의 상품에 여러 종류의 추가구성품이 포함될 수 있으므로 배열로 관리가 되며, 각각의 추가구성품 정보는 다음과 같은 구조를 가집니다.   

```javascript
{
    id : "supplement-a",    //추가구성품의 ID
    name : "추가구성품 A",    //추가구성품 상품명
    price : 1000,           //추가구성품 가격
    quantity : 1            //추가구성품 수량
}
```


### Shipping  

상품에 대해 적용될 배송비 정책을 설정합니다. 예시로, 주문 내 모든 상품을 한 택배박스로 보낼 예정이면 `naverProducts`내의 모든 Product에 대해서 `Product.Shipping.groupId`를 같은 값을 설정해야 합니다.  
때문에, `naverProducts`의 groupId가 모두 다른 값을 가지거나 공백("")의 값을 가지면 상품별 적용된 모든 배송비가 합산되어 계산됩니다.  

```javascript
{
	groupId : "GroupFopOrder",  //상품별 묶음배송구분을 위한 카테고리. groupId가 동일한 Product끼리 묶음배송 처리됩니다.
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500,  //기본 배송비
	feeRule : {
		freeByThreshold : 20000
	},
	feePayType : "PREPAYED" //PREPAYED(선불), CASH_ON_DELIVERY(착불)
}
```

## 2.2 네이버페이 구매 유입경로 설정  
### param.naverInterface   

```javascript
{
	"cpaInflowCode" : {도메인의 cookie 중 "CPAValidator" cookie값},
	"naverInflowCode" : {도메인의 cookie 중 "NA_CO" cookie값},
	"saClickId" : {도메인의 cookie 중 "NVADID" cookie값}
}
```

네이버페이 구매 유입경로 분석을 위해 네이버에서 제공되는 `//wcs.naver.net/wcslog.js` javascript를 설치하면 네이버검색 또는 네이버쇼핑 등을 통해 사이트에 진입한 구매자에 대해 도메인 쿠키영역에 `CPAValidator`, `NA_CO`, `NVADID` 값이 자동으로 설정됩니다.  

이 값을 읽어 `IMP.request_pay(param)`호출 시 `param.naverInterface`파라메터로 전달해주시면 네이버에 전달되어 분석됩니다.  

```javascript
IMP.request_pay({
	// 다른 파라메터는 생략
	naverInterface : {
		"cpaInflowCode" : {도메인의 cookie 중 "CPAValidator" cookie값},
		"naverInflowCode" : {도메인의 cookie 중 "NA_CO" cookie값},
		"saClickId" : {도메인의 cookie 중 "NVADID" cookie값}
	}
});
```

## 2.3 네이버페이 도서/공연비 추가 소득공제 주문 처리  
### param.naverCultureBenefit  

문화체육관광부에 도서/공연상품 판매 업체로 등록된 경우(사전에 네이버를 통해 등록되어야 합니다) 결제되는 주문 건이 도서/공연비 추가 공제 대상인지 여부를 지정할 수 있습니다.  

`IMP.request_pay(param)` 호출 시 `param.naverCultureBenefit = true` 파라메터를 전달해주시면 도서/공연비 추가 공제 주문으로 간주됩니다.  

```
IMP.request_pay({
    //다른 파라메터는 생략
    naverCultureBenefit : true
});
```

### 주의사항  
`naverCultureBenefit` 파라메터는 주문 단위로 관리가 되며 주문 내 상품 단위로 적용은 불가능합니다. 때문에, 도서/공연비 추가 공제 대상 상품과 비대상 상품은 혼합하여 결제할 수 없도록 사전 차단해주셔야 합니다.  



# 3. 네이버의 상품정보 체크  
`naverProducts` 파라메터와 함께 `IMP.request_pay(param, callback)`함수가 호출되면 새창이 열리면서 네이버페이 결제화면으로 이동하게 됩니다. 이동 후 하단에 있는 네이버페이 "결제하기"버튼을 누르게 되면 네이버페이 서버는 지정된 URL로 상품정보가 올바른지(Validation), 구매 가능한 재고수량은 충분한지를 체크하기 위해 HTTP GET요청을 보내게 됩니다.  

네이버페이 서버의 GET 요청에 대해 적절한 응답(xml응답)을 해야지만 `naverProducts`가 구매가능한 상태로 판단해 최종 결제 프로세스를 처리하게 됩니다.  

```
http://{지정된 URL}?product[0][id]=singleProductId&product[1][id]=optionProductId&product[1][optionManageCodes]=R_L
```

이 때, xml문서의 정보가 불충분 또는 잘못되었거나 `naverProducts`의 내용과 일치하지 않는 내용이 확인되면 아래와 같이 구매불가 팝업이 나타나 결제진행이 이뤄지지 않습니다.  

![](screenshot/naver-wrong-xml.png)


## 3.1 Query String  

위 예시의 경우, 구매자가 선택한 옵션이 없는 상품(singleProductId)와 옵션이 있는 상품(optionProductId) 2종류의 상품 구매가 진행되었기 때문에 상품정보를 요청하는 query string이 다음과 같이 전달됩니다.  

### 선택된 옵션이 없는 상품  

- product[0][id] : singleProductId (`Product.id`를 의미)

### 옵션이 선택된 상품  

- product[1][id] : optionProductId (`Product.id`를 의미)
- product[1][optionManageCode] : R_L(`Product.option.selectionCode`를 의미)


## 3.2 xml 응답  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<products>
	<!-- 선택된 옵션이 없는 상품의 상세 정보 -->
	<product>
      <id>singleProductId</id>
      <name>상품singleProduct</name>
      <basePrice>1000</basePrice>
      <taxType />
      <infoUrl>http://www.iamport.kr/product/detail</infoUrl>
      <imageUrl>http://www.iamport.kr/product/detail/thumbnail</imageUrl>
      <status>ON_SALE</status>
      <shippingPolicy>
         <groupId />
         <method>DELIVERY</method>
         <feeType>FREE</feeType>
         <feePayType>FREE</feePayType>
         <feePrice>0</feePrice>
      </shippingPolicy>
   </product>
   
   <!-- 옵션이 선택된 상품 -->
   <product>
      <id>optionProductId</id>
      <name>상품optionProduct</name>
      <basePrice>1000</basePrice>
      <taxType />
      <infoUrl>http://www.iamport.kr/product/detail</infoUrl>
      <imageUrl>http://www.iamport.kr/product/detail/thumbnail</imageUrl>
      <status>ON_SALE</status>
      <shippingPolicy>
         <groupId />
         <method>DELIVERY</method>
         <feeType>FREE</feeType>
         <feePayType>FREE</feePayType>
         <feePrice>0</feePrice>
      </shippingPolicy>
      <optionSupport>true</optionSupport>
      <option>
      	  <!-- 상품이 제공하는 모든 옵션 정보를 나열 -->
         <optionItem>
            <type>SELECT</type>
            <name>색상</name>
            <value>
               <id>RED</id>
               <text>빨강</text>
            </value>
            <value>
               <id>BLUE</id>
               <text>파랑</text>
            </value>
            <value>
               <id>GREEN</id>
               <text>초록</text>
            </value>
         </optionItem>
         <optionItem>
            <type>SELECT</type>
            <name>사이즈</name>
            <value>
               <id>150</id>
               <text>150</text>
            </value>
            <value>
               <id>160</id>
               <text>160</text>
            </value>
            <value>
               <id>170</id>
               <text>170</text>
            </value>
            <value>
               <id>180</id>
               <text>180</text>
            </value>
         </optionItem>
         
         <!-- 옵션 타입(optionItem)이 2가지 이상인 경우, 조합될 수 있는 경우의 수를 모두 나열 -->
         <!-- 옵션 타입(optionItem)이 1가지더라도, 옵션별로 가격이 달라질 수 있으면 모두 나열 -->
         <combination>
            <manageCode>R_S</manageCode>
            <options>
               <name>색상</name>
               <id>RED</id>
            </options>
            <options>
               <name>사이즈</name>
               <id>160</id>
            </options>
         </combination>
         <combination>
            <manageCode>R_M</manageCode>
            <price>100</price>
            <options>
               <name>색상</name>
               <id>RED</id>
            </options>
            <options>
               <name>사이즈</name>
               <id>170</id>
            </options>
         </combination>
         <combination>
            <manageCode>R_L</manageCode>
            <price>200</price>
            <options>
               <name>색상</name>
               <id>RED</id>
            </options>
            <options>
               <name>사이즈</name>
               <id>180</id>
            </options>
         </combination>
      </option>
   </product>
</products>
```

## 3.3 option 이해하기  
구매자가 선택할 옵션이 있는 상품(예시의 optionProductId)의 경우, xml 응답을 통해 상품정보를 확인하는 과정에서 "구매불가"처리되어 중단되는 경우를 연동하는 단계에서 자주 확인할 수 있었습니다. 예시를 통해 대표적인 사례를 확인해보도록 하겠습니다.  

옵션이 포함된 상품의 경우, XML 응답에서 아래와 같이 `optionSupport : true` 가 추가되어야 합니다.  

```xml
<product>
    <optionSupport>true</optionSupport>
    <!-- 다른 Node는 생략 -->
</product>
```

### 3.3.1 선택가능한 옵션이 2가지 이상인 경우  
앞의 예시처럼, 신발 구매를 위해 색상-옵션과 사이즈-옵션을 모두 선택해야 구매가 가능한 상품의 경우 `옵션조합`에 대한 정보가 필수로 제공되어야 합니다.  

#### 3.3.1.a `IMP.request_pay()` 파라메터 예시  

`naverProducts.Product.option` 객체는 다음과 같은 값을 가집니다. **이 경우, 반드시 특정 옵션조합을 의미하는 `selectionCode`를 지정해야 합니다.(이 값은 `3.3.1.b`에서 `combination/manageCode`와 연결됩니다.)**    

색상은 빨강, 사이즈는 180mm 옵션을 각각 선택한 것을 의미하며 

```javascript
{
	optionPrice : 200,
	selectionCode : "R_L",
	selections : [
		{
			code : "RED",
			label : "색상",
			value : "빨강"
		},
		{
			code : "180",
			label : "사이즈",
			value : "180"
		}
	]
}
```


#### 3.3.1.b xml 응답 예시  

상품ID가 `optionProductId`인 상품이 가질 수 있는 옵션의 종류는 `<optionItem>`로 표현되며 `<option>`노드 하위에 포함되어야 합니다.  

색상, 사이즈를 의미하는 2개의 optionItem으로 구성되어 있으며 색상 optionItem에는 `RED`옵션이 포함되어있고, 사이즈 optionItem에는  `180`옵션이 포함되어있습니다.  

```xml
<option>
	<optionItem>
        <type>SELECT</type>
        <name>색상</name>
        <value>
           <id>RED</id>
           <text>빨강</text>
        </value>
        <value>
           <id>BLUE</id>
           <text>파랑</text>
        </value>
        <value>
           <id>GREEN</id>
           <text>초록</text>
        </value>
     </optionItem>
     <optionItem>
        <type>SELECT</type>
        <name>사이즈</name>
        <value>
           <id>150</id>
           <text>150</text>
        </value>
        <value>
           <id>160</id>
           <text>160</text>
        </value>
        <value>
           <id>170</id>
           <text>170</text>
        </value>
        <value>
           <id>180</id>
           <text>180</text>
        </value>
     </optionItem>
     
     <!-- 이하 combination 생략 -->
</option>
```

또한, 2가지 옵션이 조합되었으므로 옵션조합을 구분하는 `<combination>`노드 또한 `<option>`노드 하위에 포함되어야 합니다.  

앞에서 나열한 2개의 `optionItem`중 `RED`와 `180`을 조합한 경우를 설명하고 있으며, 이 조합은 상품기본가격에 **200**원이 추가되는 것으로 표현되어있습니다.  

```xml
<option>
	<!-- 이상 optionItem 생략 -->
	<combination>
		<manageCode>R_L</manageCode>
		<price>200</price>
		<options>
			<name>색상</name>
			<id>RED</id>
		</options>
		<options>
			<name>사이즈</name>
			<id>180</id>
		</options>
	</combination>
</option>
```

### 3.3.2 선택 가능한 옵션이 1가지인 경우  

#### 3.3.2.a `IMP.request_pay()` 파라메터 예시  

- 1가지의 옵션만을 선택하기 때문에 `selectionCode`파라메터는 제거되어야 합니다.  
- 또한, 이 경우는 옵션에 대한 추가 금액을 지정하는 것이 불가능합니다.  

```javascript
{
	selections : [
		{
			code : "180",
			label : "사이즈",
			value : "180"
		}
	]
}
```

#### 3.3.2.b xml 응답 예시  

1가지 옵션만 제공되는 상품이라면, `combination`노드는 필요가 없습니다. 때문에, 다음과 같이 `optionItem`노드만 정의되면 됩니다.  

```xml
<option>
	<optionItem>
        <type>SELECT</type>
        <name>색상</name>
        <value>
           <id>RED</id>
           <text>빨강</text>
        </value>
        <value>
           <id>BLUE</id>
           <text>파랑</text>
        </value>
        <value>
           <id>GREEN</id>
           <text>초록</text>
        </value>
     </optionItem>
     <optionItem>
        <type>SELECT</type>
        <name>사이즈</name>
        <value>
           <id>150</id>
           <text>150</text>
        </value>
        <value>
           <id>160</id>
           <text>160</text>
        </value>
        <value>
           <id>170</id>
           <text>170</text>
        </value>
        <value>
           <id>180</id>
           <text>180</text>
        </value>
     </optionItem>
     
     <!-- 이하 combination 생략 -->
</option>
```

만약, `3.3.1.b`와 같이 `optionItem` 노드와 `combination` 노드가 모두 정의되어있으면 옵션을 1가지만 선택하고 상품을 구매하는 것도 가능해지고, 2가지 선택하는 것도 가능하게 됩니다.  


### 3.3.3 선택가능한 옵션이 1가지 이지만, 옵션값에 따라 가격이 변동되는 경우  

옵션별 추가금액을 적용하기 위해서는 `combination`노드의 정의가 필요합니다. 때문에, `3.3.1`과 마찬가지로 `optionItem`노드 뿐 아니라 `combination`노드가 동시에 정의되어야 합니다.  

#### 3.3.3.a `IMP.request_pay()` 파라메터 예시  

옵션에 대한 추가금액을 의미하는 `optionPrice`속성이 선언되어야 하며 `combination` 노드와 매칭할 수 있도록 `selectionCode`가 정의되어야 합니다.  

```javascript
{
	optionPrice : 200,
	selectionCode : "Large",
	selections : [
		{
			code : "180",
			label : "사이즈",
			value : "180"
		}
	]
}
```

#### 3.3.3.b xml 응답 예시  

(반복되는 내용이므로 optionItem 노드에 대해서는 설명을 생략합니다.)  
추가금액이 지정된 옵션에 대한 `combination`노드를 정의해야 합니다. `naverProducts`에서 Large 라는 `selectionCode`를 선언했기 때문에 manageCode가 Large인 `combination`노드가 정의되어있어야 합니다.   

```xml
<option>
	<!-- 이상 optionItem 생략 -->
	<combination>
		<manageCode>Large</manageCode>
		<price>200</price>
		<options>
			<name>사이즈</name>
			<id>180</id>
		</options>
	</combination>
</option>
```

## 3.4 supplement 이해하기  

구매상품에 추가구성품이 포함되어 구매되는 경우, XML 응답에서 아래와 같이 `supplementSupport : true` 가 추가되어야 합니다.  

```xml
<product>
    <supplementSupport>true</supplementSupport>
    <!-- 다른 Node는 생략 -->
</product>
```

### 3.4.1 `IMP.request_pay()` 예시  

아래와 같이 `naverProducts` 속성으로 상품에 대한 object 가 전달되었다고 하면, `singleProductId` 상품 1개와 추가구성품 2개 `supplement-a`, `supplement-b`를 함께 구매하는 경우입니다.  

```javascript
{
    id : "singleProductId",
    name : "네이버페이 상품1",
    basePrice : 1000,
    //다른 프로퍼티 생략
    supplements : [
        {
            id : "supplement-a",
            name : "추가구성품 A",
            price : 1000, //1개당 구매 가격
            quantity : 1
        },
        {
            id : "supplement-b",
            name : "추가구성품 B",
            price : 1200, //1개당 구매 가격
            quantity : 2
        }
    ]
}
```

### 3.4.2 xml 응답 예시  

`IMP.request_pay()` 호출 시 3.4.1과 같이 호출하였다면, 네이버페이 서버로부터 요청된 상품정보 XML 응답에는 아래와 같이 supplement node가 2개 추가된 상태로 응답을 하시면 됩니다.   

```xml
<product>
    <id>singleProductId</id>
    <name>상품singleProduct</name>
    <basePrice>1000</basePrice>
    <!-- 다른 Node는 생략 -->
    <supplementSupport>true</supplementSupport>
    <supplement>
        <id>supplement-a</id>
        <name>추가구성품 A</name>
        <price>1000</price> <!-- 1개당 구매 가격 -->
        <stockQuantity>24</stockQuantity> <!-- 현재 남아있는 구매가능 수량 재고 -->
        <status>true</status> <!-- 추가구성품 판매 가능상태이면 true -->
    </supplement>
    <supplement>
        <id>supplement-b</id>
        <name>추가구성품 B</name>
        <price>1200</price> <!-- 1개당 구매 가격 -->
        <stockQuantity>24</stockQuantity> <!-- 현재 남아있는 구매가능 수량 재고 -->
        <status>true</status> <!-- 추가구성품 판매 가능상태이면 true -->
    </supplement>
</product>
```


## 4. 결제정보 동기화   
주문상품 연동형태의 네이버페이 결제는 결제가 완료된 직후에 완료 여부를 가맹점 혹은 아임포트가 알 수 없는 구조로 구성되어있습니다.  
결제정보 동기화를 위해서는 배치(Batch)작업으로 결제변동내역을 조회하는 API를 요청해야하며, 아임포트 서버가 변동 결제내역을 확인하여 주문상태를 변경하는 작업을 주기적으로 수행하고 있습니다.  


### 순서  
1. `IMP.request_pay(param, callback)`호출
2. 네이버페이 결제창을 통해 결제 프로세스 진행
3. (아임포트 내에서는` 미결제` 상태로 주문정보가 생성된 상태)
4. 구매자가 결제 프로세스를 완료(결제 성공)
5. 아임포트 배치 서버가 결제완료여부를 체크(1분단위로 배치 수행)
6. **아임포트 거래 건의 결제상태를 `paid`로 변경 및 주문 관련정보 동기화**
7. 결제완료를 알리는 Notification URL통지

### 주문 정보의 동기화    
네이버페이의 경우 일반 PG결제수단과 달리 `주문자 이름`, `주문자 전화번호`, `배송주소` 등의 정보가 네이버페이 결제단계에서 입력 및 확정됩니다.  
또한, 조건별 배송비 부과 정책에 따라 실제 결제되는 금액이 결제단계에서 달라질 수 있습니다.  

때문에, `IMP.request_pay(param)` 호출 시점에 전달되는 아래 파라메터는 실제 결제완료 후 동기화 단계에서 `보정(overwrite)`됩니다. 

- amount
- buyer_name
- buyer_email
- buyer_tel
- buyer_addr
- buyer_postcode

### 4.1 네이버페이 결제 시작  

아래의 코드와 같이 네이버페이 결제창을 호출하면, [아임포트 관리자 페이지](https://admin.iamport.kr)의 결제승인내역 목록에 결제 정보가 아래와 같이 먼저 기록됩니다.  

- amount : 14000
- buyer_email : 'iamport@siot.do'
- buyer_name : '구매자이름'
- buyer_tel : '010-1234-5678'
- buyer_addr : '서울특별시 강남구 삼성동'
- buyer_postcode : '123-456'

```javascript
IMP.request_pay({
    pg : 'naverco',
    pay_method : 'card', //연동되지 않습니다. 네이버페이 결제창 내에서 결제수단을 구매자가 직접 선택하게 됩니다.
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', //누락되면 이니시스 결제창에서 오류
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverProducts : [
    	{
    		id : "singleProductId",
    		name : "네이버페이 상품1",
    		basePrice : 14000,
    		taxType : 'TAX_FREE', //TAX or TAX_FREE
    		quantity : 1,
    		infoUrl : "http://www.iamport.kr/product/detail",
    		imageUrl : "http://www.iamport.kr/product/detail/thumbnail",
    		shipping : {
    			groupId : "shipping-a",
    			method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
    			baseFee : 2500,
    			feeRule : {
    				surchargesByArea : [ //array or string(API)
    					{area:"island", surcharge:2000},
    					{area:"jeju", surcharge:4500}
    				]
    			},
    			feePayType : "PREPAYED" //PREPAYED(선불), CASH_ON_DELIVERY(착불)
    		}
    	}
	]
});
```

### 4.2 구매자가 배송받을 지역을 "제주"로 설정하여 결제한 경우   

위 설정에 따라 실제 구매자가 결제한 금액은 상품금액 14000원 + 제주지역 추가배송비 4500원 = 18500원이 됩니다.  
또한, 최초 전달된 "서울특별시 강남구 삼성동"이라는 주소 대신 "제주시 첨단로 123" 주소가 기입되었기 때문에 해당 정보도 변경이 필요합니다.  

이에 따라 

- amount : 14000 -> 18500
- buyer_addr : "서울특별시 강남구 삼성동" -> "제주시 첨단로 123"

으로 변경되며, [아임포트 관리자 페이지](https://admin.iamport.kr)의 결제 승인내역 및 REST API로 해당 건에 대해 상세정보 조회를 하면 변경된 정보가 응답됩니다.  

네이버페이 결제단계에서 구매자의 이름, 전화번호 등이 변경되는 경우에도 동일하게 아임포트 거래내역에 변경되어 저장되며 변경이 가능한 필드 목록은 4.1에서 나열한 총 6가지 필드입니다.  


# 5. 찜하기 동작  

네이버페이 찜하기 버튼이 클릭되었을 때, 네이버페이에 상품과 관련된 정보가 전송되어야 합니다.  
아임포트 javascript API를 통해 해당 기능을 쉽게 구현하실 수 있습니다. (iamport.payment-1.1.6.js부터 지원)

```javascript
IMP.naver_zzim({
	naverProducts: [
		{
			"id" : "상품아이디",
			"name" : "상품명",
			"desc" : "상품상세설명",
			"uprice" : 10000, //상품가격
			"url" : "http://www.yourshop.com/product/123", //상품고유 URL
			"image" : "http://www.yourshop.com/product/123/thumbnail" //상품 대표 썸네일이미지 URL
		},
		{
			"id" : "상품아이디",
			"name" : "상품명",
			"desc" : "상품상세설명",
			"uprice" : 10000, //상품가격
			"url" : "http://www.yourshop.com/product/123", //상품고유 URL
			"image" : "http://www.yourshop.com/product/123/thumbnail" //상품 대표 썸네일이미지 URL
		}
	]
});
```

찜한 상품이 추후 결제로 이어질 수 있도록 `상품아이디(id)`는 네이버페이 결제 시 전달하는 `naverProduducts` 의 `상품아이디(id)` 와 일치시켜주는 것이 좋습니다.  