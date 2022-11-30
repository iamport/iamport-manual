# 주문형 네이버페이 연동하기

:globe_with_meridians: [EN](/en/NAVERPAY/sample/naverpay-order.md)  

주문형은 가맹점 사이트에서 주문 단계부터 상품정보와 연동하여 네이버페이로 결제를 진행하는 방식입니다.

자세한 연동 가이드는 아래 문서를 참고해주세요

<a href="https://s3.us-west-2.amazonaws.com/secure.notion-static.com/c61e773a-6f2c-4744-8976-6fef7f839da3/%EB%84%A4%EC%9D%B4%EB%B2%84%ED%8E%98%EC%9D%B4%28%EC%A3%BC%EB%AC%B8%ED%98%95%29_Naver_Pay-Shop_Interworking_Guide_2.1-ko_%281%29.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Content-Sha256=UNSIGNED-PAYLOAD&X-Amz-Credential=AKIAT73L2G45EIPT3X45%2F20221130%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20221130T021917Z&X-Amz-Expires=86400&X-Amz-Signature=1d899ba7629ae44fc16d6ee56561d10f37d821a7b5cf9a77cca529725b47412f&X-Amz-SignedHeaders=host&response-content-disposition=filename%3D%22%25EB%2584%25A4%25EC%259D%25B4%25EB%25B2%2584%25ED%258E%2598%25EC%259D%25B4%28%25EC%25A3%25BC%25EB%25AC%25B8%25ED%2598%2595%29_Naver_Pay-Shop_Interworking_Guide_2.1-ko%2520%281%29.pdf%22&x-id=GetObject" target="_blank"> 공식 연동가이드 확인 </a>

## 1. PG 설정하기

<a href="https://guide.iamport.kr/485c6da8-01d7-4900-bc05-76005e5477ba" target="_blank">네이버페이(주문형/결제형) 테스트 모드 설정</a> 페이지에서 **1) 주문형** 내용을 참고하여 PG 설정을 합니다.

## 2. 네이버페이 버튼 추가하기<a id="add-button"></a>

네이버페이 구매하기 기능은 네이버페이가 제공하는 SDK를 통해 지정된 디자인을 그대로 사용해 구현해야 합니다.  

![](screenshot/naver-button.png)

**네이버페이 버튼 UI생성을 위한 SDK** 스크립트를 다음과 같이 페이지에 추가합니다.  

```html
<!-- PC용 -->
<script type="text/javascript" src="https://pay.naver.com/customer/js/naverPayButton.js"></script>

<!-- 모바일용 -->
<script type="text/javascript" src="https://pay.naver.com/customer/js/mobile/naverPayButton.js"></script>
```
`naver.NaverPayButton.apply` 함수를 호출하여 지정한 HTML element를 네이버페이 버튼 UI로 변환합니다.  

다음 버튼 클릭 핸들러 속성을 정의하여 버튼 클릭 액션을 구현합니다.
- `BUY_BUTTON_HANDLER` : **NPay구매버튼** 클릭 핸들러 속성
   - [아임포트 네이버페이 결제창 호출 함수 호출](#request-pay)
- `WISHLIST_BUTTON_HANDLER` : **찜버튼** 클릭 핸들러 속성
   - [아임포트 네이버페이 찜하기 함수 호출](#zzim)
  
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
		//핸들러 내에서 결제창 호출 함수 호출
		IMP.request_pay(param);
	},
	WISHLIST_BUTTON_HANDLER : function() {
		//중략
		//핸들러 내에서 찜하기 함수 호출
		IMP.naver_zzim(param);
	}
});
```

## 3. 결제창 호출하기<a id="request-pay"></a>

[IMP.request_pay(param)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 네이버페이 결제창을 호출합니다.

네이버페이는 결제 완료 여부가 즉시 통지되지 않기 때문에, 결제가 완료된 시점에 `IMP.request_pay(param)`의 callback 함수나 `m_redirect_url`이 호출되지 않습니다. *단, 네이버페이 결제 프로세스가 시작되기 전에 데이터 검증 등에 실패하는 경우에는 callback 함수가 호출됩니다.*  

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `naverco`로 지정합니다.
- `pay_method` : 호출 시 선택된 값은 무시되며, 네이버페이 결제창에서 선택한 옵션으로 설정됩니다.
- `buyer_tel` : 필수 입력.
- `naverCultureBenefit` : 도서/공연비 추가 공제 주문 여부 (true/false).
    - 문화체육관광부에 도서/공연상품 판매 업체로 등록 및 사전 네이버를 통한 등록을 해야 적용할 수 있습니다.
    - 주문 단위로 적용되므로 공제 대상 상품과 비대상 상품을 혼합하여 결제할 수 없습니다. 
- `naverInterface` : 네이버페이 구매에 대한 유입경로 분석을 위해 네이버페이에 전달하는 정보를 설정합니다.
    - 네이버페이 구매 유입경로 분석을 위해 네이버에서 제공되는 `//wcs.naver.net/wcslog.js` JavaScript를 설치하면 네이버검색 또는 네이버쇼핑 등을 통해 사이트에 진입한 구매자에 대해 도메인 쿠키영역에 `CPAValidator`, `NA_CO`, `NVADID` 값이 자동으로 설정됩니다. 해당 값을 읽어 설정합니다.
- `naverProducts` : [Product](#product) 객체 배열로 구성된 상품정보(필수 입력).  
    - [naverProducts JSON Schema](naverpay-schema.md)


```javascript
IMP.request_pay({
    pg : 'naverco',
    pay_method : 'card',
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    naverInterface : {
		"cpaInflowCode" : {도메인의 cookie 중 "CPAValidator" cookie값},
		"naverInflowCode" : {도메인의 cookie 중 "NA_CO" cookie값},
		"saClickId" : {도메인의 cookie 중 "NVADID" cookie값}
	},
    naverProducts : [
    	{
    		id : "singleProductId", //선택된 옵션이 없는 상품
    		name : "네이버페이 상품1",
    		basePrice : 1000,
    		taxType : 'TAX_FREE', //TAX or TAX_FREE
    		quantity : 2,
    		infoUrl : "http://www.iamport.kr/product/detail",
    		imageUrl : "http://www.iamport.kr/product/detail/thumbnail",
         giftName : "사은품A",
    		shipping : {
    			groupId : "shipping-a",
    			method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
    			baseFee : 2500,
    			feeRule : {
    				freeByThreshold : 20000
    			},
    			feePayType : "PREPAYED" //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
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
    		id : "optionProductId", //옵션이 선택된 상품
    		name : "네이버페이 상품2",
    		basePrice : 1000,
    		taxType : 'TAX_FREE', //TAX or TAX_FREE
    		infoUrl : "http://www.iamport.kr/product/detail",
    		imageUrl : "http://www.iamport.kr/product/detail/thumbnail",
         giftName : "사은품B",
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
				feePayType : "PREPAYED" //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
			}
		}
	]
}, function(rsp) {
    if ( !rsp.success ) {
    	//결제 시작되기 전에 오류가 난 경우
        var msg = '오류로 인하여 결제가 시작되지 못하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

### `naverProducts` 배열의 Product 객체<a id="product"></a>

구매할 개별 상품에 대한 상세 정보와 다음 추가 정보로 구성됩니다.  

- `options` (객체 배열): 상품에 대해 선택한 한개 이상의 옵션들
- `supplements` (객체 배열): 한개 이상의 추가구성품에 대한 상세 정보
- `shipping` (객체): 적용될 배송비 정책  

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
   "giftName" : "사은품A", //해당상품 구매시 제공되는 사은품 명칭(없으면 정의하지 않음)
	"options" : "array(of option)",     //구매자가 선택한 상품 옵션에 대한 상세 정보
   "supplements" : "array(of supplement)", //추가구성품에 대한 상세 정보
	"shipping" : "object(of shipping)"    //상품 배송관련 상세 정보
}
```

### `options` 배열의 Option 객체<a id="option"></a>

구매자가 **선택한 옵션**에 대한 상세 내용을 설정합니다. 다음은 RED 색상과 180mm 사이즈를 선택한 예시입니다.

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

### `supplements` 배열의 Supplement 객체<a id="supplement"></a>

상품과 연관된 추가구성품를 포함하여 구매하는 경우, 추가구성품에 대한 상세 정보를 설정합니다.  

```javascript
{
    id : "supplement-a",    //추가구성품의 ID
    name : "추가구성품 A",    //추가구성품 상품명
    price : 1000,           //추가구성품 가격
    quantity : 1            //추가구성품 수량
}
```


### `shipping` 객체<a id="shipping"></a>

상품에 대해 적용될 배송비 정책을 설정합니다. 자세한 내용은 [네이버페이 배송비 정책 및 예시](naverpay-shipping.md)를 참고하세요.  
- 주문 내 모든 상품 묶음배송 처리: `naverProducts`의 모든 Product 객체에 동일한 `shipping.groupId`를 설정합니다.  
- 상품별 개별배송 처리: `naverProducts`의 모든 Product 객체에 다른 `shipping.groupId` 또는 공백("")의 값을 설정합니다. *상품별 적용된 모든 배송비가 합산되어 계산됩니다.*  

```javascript
{
	groupId : "GroupFopOrder",  //상품별 묶음배송구분을 위한 카테고리. groupId가 동일한 Product끼리 묶음배송 처리됩니다.
	method : "DELIVERY", //DELIVERY(택배·소포·등기), QUICK_SVC(퀵 서비스), DIRECT_DELIVERY(직접 전달), VISIT_RECEIPT(방문 수령), NOTHING(배송 없음)
	baseFee : 2500,  //기본 배송비
	feeRule : {
		freeByThreshold : 20000
	},
	feePayType : "PREPAYED" //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
}
```


## 4. 네이버페이의 상품정보 검증하기  

네이버페이 결제창 하단의 네이버페이 **결제하기** 버튼을 누르면 네이버페이 서버는 가맹점이 네이버페이에 등록한 URL로 HTTP GET 요청을 보내 상품정보 및 구매 가능한 재고수량을 검증합니다.   

네이버페이 서버의 GET 요청에 대해 적절한 응답(XML)을 해야지만 해당 주문의 상품들이 구매가능한 상태로 검증되어 최종 결제 프로세스가 처리됩니다.  

XML 응답의 정보가 불충분 또는 잘못되었거나 요청한 상품정보(`naverProducts`)의 내용과 일치하지 않는 경우 아래와 같이 구매불가 팝업이 나타나고 결제진행이 중단됩니다.  

![](screenshot/naver-wrong-xml.png)
  
## 4.1 상품정보 검증 요청  

네이버페이는 가맹점에 HTTP GET 요청을 보내 상품정보를 검증합니다. 다음 2종류의 상품 구매 시 검증에 필요한 상품정보는 다음과 같이 query string으로 전달됩니다.  

- 선택된 옵션이 없는 상품 :  
    - product[0][id] : singleProductId (`Product.id` 값)
- 옵션이 선택된 상품 : 
    - product[1][id] : optionProductId (`Product.id` 값)
    - product[1][optionManageCode] : R_L(`Product.option.selectionCode` 값)



```
http://{네이버페이에 등록된 URL}?product[0][id]=singleProductId&product[1][id]=optionProductId&product[1][optionManageCodes]=R_L
```
네이버페이에서 가맹점으로 요청하는 상품정보XML URL 중 '가맹점연동도메인/가맹점페이지' 영역은 가맹점에서 임의로 설정하실 수 있으며,
상품정보XML 이 응답되도록 구현하신 후 'dl_techsupport@navercorp.com' 로 전달주시면 가맹점 정보에 등록해드리고 있습니다.
(※ 별도의 요청이 없으면 기본적으로 'http://가맹점연동도메인/?wc-api=naver-product-info' 로 연동됩니다.)

## 4.2 XML 응답<a id="xml-response"></a>

[네이버페이 상품 옵션과 추가구성품에 대한 요청 및 응답 예시](naverpay-validation.md)를 참고하세요.

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
      <giftName>사은품A</giftName>
      <status>ON_SALE</status>
      <shippingPolicy>
         <groupId />
         <method>DELIVERY</method>
         <feeType>FREE</feeType>
         <feePayType>FREE</feePayType>
         <feePrice>0</feePrice>
      </shippingPolicy>
   </product>

   <!-- 옵션이 선택된 상품의 상세 정보 -->
   <product>
      <id>optionProductId</id>
      <name>상품optionProduct</name>
      <basePrice>1000</basePrice>
      <taxType />
      <infoUrl>http://www.iamport.kr/product/detail</infoUrl>
      <imageUrl>http://www.iamport.kr/product/detail/thumbnail</imageUrl>
      <giftName>사은품B</giftName>
      <status>ON_SALE</status>
      <shippingPolicy>
         <groupId />
         <method>DELIVERY</method>
         <feeType>FREE</feeType>
         <feePayType>FREE</feePayType>
         <feePrice>0</feePrice>
      </shippingPolicy>
      <optionSupport>true</optionSupport> <!-- 옵션이 선택된 상품의 경우, 필수 설정 -->
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
         </optionItem>
         <optionItem>
            <type>SELECT</type>
            <name>사이즈</name>
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
         <!-- ...생략(RED와 동일하게 BLUE 색상에 대해 조합 정의)... -->
      </option>
   </product>
</products>
```


## 5. 결제정보 동기화하기   

주문형은 결제완료 통지가 즉시 이루어지지 않기 때문에 아임포트 서버는 주기적으로 결제변동내역을 확인하는 배치작업을 수행하여 주문상태를 동기화합니다.  

### 결제 프로세스 순서 및 상태  
1. 네이버페이 결제창을 호출하여 결제 프로세스 진행(`미결제` 상태)
2. 구매자가 결제 프로세스를 완료(결제 성공)
3. 아임포트 배치 서버가 결제완료여부를 체크(1분단위로 배치 수행)
4. **아임포트 거래 건의 결제상태를 `paid`로 변경 및 주문 관련정보 동기화**
5. 결제완료를 알리는 Notification URL 통지

### 주문 정보의 동기화  

네이버페이는 일반 PG결제수단과 달리 구매자 정보, 배송지 및 배송비 정보가 네이버페이 결제단계에서 입력 및 확정됩니다.   

#### 예시

다음과 같이 네이버페이 결제창을 호출하면, [아임포트 관리자 페이지](https://admin.iamport.kr)의 결제승인내역 목록에 결제 정보가 아래와 같이 먼저 기록됩니다. *다음은 결제단계에서 변경 가능한 파라미터 목록입니다.*

- amount : 14000
- buyer_email : 'iamport@siot.do'
- buyer_name : '구매자이름'
- buyer_tel : '010-1234-5678'
- buyer_addr : '서울특별시 강남구 삼성동'
- buyer_postcode : '123-456'
  
결제단계에서 구매자가 선택한 상품 배송정보에 따라 다음과 같이 결제정보가 변경됩니다. [아임포트 관리자 페이지](https://admin.iamport.kr)의 결제승인내역 및 REST API로 해당 건에 대한 변경된 정보를 확인할 수 있습니다. 

- amount : 14000 -> 18500
- buyer_addr : "서울특별시 강남구 삼성동" -> "제주시 첨단로 123"

```javascript
IMP.request_pay({
    pg : 'naverco',
    pay_method : 'card', // 네이버페이 결제창에서 선택한 결제수단으로 overwrite됨.
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678', // 필수 입력
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
         giftName : "사은품A",
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
    			feePayType : "PREPAYED" //PREPAYED(선불) 또는 CASH_ON_DELIVERY(착불)
    		}
    	}
	]
});
``` 



## 6. 찜하기<a id="zzim"></a>  

[네이버페이 찜하기 버튼](#add-button) 핸들러에서 다음과 같이 아임포트 JavaScript 함수를 호출하여 네이버페이에 찜한 상품정보를 전송합니다.(iamport.payment-1.1.6.js부터 지원)  

찜한 상품이 추후 결제로 이어질 수 있도록 상품아이디(`naverProduducts.id`)는 네이버페이 결제 시 전달하는 상품아이디(`naverProduducts.id`)와 동일하게 설정합니다.  

```javascript
IMP.naver_zzim({
  naverProducts: [
    {
      "id" : "상품아이디",
      "name" : "상품명",
      "desc" : "상품상세설명",
      "uprice" : 10000, //상품가격
      "url" : "http://www.yourshop.com/product/123", //상품고유 URL
      "thumb" : "http://www.yourshop.com/product/123/thumbnail", //상품 대표 썸네일이미지 URL
      "image" : "http://www.yourshop.com/product/123/image" //상품 대표 메인이미지 URL
    },
    {
      "id" : "상품아이디",
      "name" : "상품명",
      "desc" : "상품상세설명",
      "uprice" : 10000, //상품가격
      "url" : "http://www.yourshop.com/product/123", //상품고유 URL
      "thumb" : "http://www.yourshop.com/product/123/thumbnail", //상품 대표 썸네일이미지 URL
      "image" : "http://www.yourshop.com/product/123/image" //상품 대표 메인이미지 URL
    }
  ]
});
```
