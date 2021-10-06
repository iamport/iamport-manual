# 네이버페이 상품 옵션과 추가구성품에 대한 요청 및 응답 예시

- [1. 상품 옵션](#product-option)
   - [1.1 선택 가능한 옵션이 1가지인 경우](#product-option1)
   - [1.2 선택 가능한 옵션이 1가지인 경우(옵션별 가격변동 적용)](#product-option1-var)
   - [1.3 선택 가능한 옵션이 2가지 이상인 경우](#product-option2)
- [2. 추가구성품](#supplement)
  
**Request**(요청)는 IMP.request_pay(param) 호출 시 전달하는 파라미터 설정입니다.  
**Response**(응답)는 네이버페이의 상품정보 검증 요청에 대한 XML 응답이며 *상품이 제공하는 모든 옵션, 추가구성품, 및 옵션 조합을 포함합니다*.

## 1. 상품 옵션<a id="product-option"/>
 
## 1.1 선택 가능한 옵션이 1가지인 경우<a id="product-option1"/>

### Request

- 옵션에 대한 추가 금액을 지정할 수 없습니다.  

```javascript
// param.naverProducts.Product.Option
{
   //다른 속성 생략
	selections : [
		{
			code : "180",
			label : "사이즈",
			value : "180"
		}
	]
}
```

### Response  

```xml
<optionSupport>true</optionSupport> <!-- 옵션이 선택된 상품의 경우, 필수 설정 -->
<option>
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
</option>
```  


## 1.2 선택 가능한 옵션이 1가지인 경우(옵션별 가격변동 적용)<a id="product-option1-var"/>

### Request  

- 옵션별 추가금액을 적용하기 위해서 `optionPrice`와 `selectionCode`를 지정합니다.  

```javascript
// param.naverProducts.Product.Option
{
	optionPrice : 200, //옵션 추가금액
	selectionCode : "Large", //옵션 조합(응답의 combination/manageCode와 대응하는 값)
	selections : [
		{
			code : "180",
			label : "사이즈",
			value : "180"
		}
	]
}
```

### Response 

```xml
<optionSupport>true</optionSupport> <!-- 옵션이 선택된 상품의 경우, 필수 설정 -->
<option>
	<!-- 이상 optionItem 생략 -->
	<combination> <!-- 추가금액이 지정된 옵션에 대한 정보 -->
		<manageCode>Large</manageCode> <!-- 요청의 selectionCode와 대응하는 값 -->
		<price>200</price> <!-- 추가금액 -->
		<options>
			<name>사이즈</name>
			<id>180</id>
		</options>
	</combination>
   <!-- ...생략(추가금액이 지정된 모든 옵션에 대해 combination 노드 정의)... -->
</option>
```

## 1.3 선택가능한 옵션이 2가지 이상인 경우<a id="product-option2"/>

### Request  

```javascript
// param.naverProducts.Product.Option
{
   optionPrice : 200, //옵션 추가금액
	selectionCode : "R_L", //옵션 조합(응답의 combination/manageCode에 대응하는 값)
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

### Response  

```xml
<optionSupport>true</optionSupport> <!-- 옵션이 선택된 상품의 경우, 필수 설정 -->
<option>
	<optionItem> <!-- 옵션 1 -->
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
   </optionItem> <!-- 옵션 2 -->
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
   <combination>
      <manageCode>R_L</manageCode> <!-- 요청의 selectionCode에 대응하는 값 -->
      <price>200</price> <!-- 추가금액 -->
      <options>
         <name>색상</name>
         <id>RED</id>
      </options>
      <options>
         <name>사이즈</name>
         <id>180</id>
      </options>
   </combination>
   <!-- ...생략(가능한 모든 옵션 조합에 대해 combination 노드 정의)... -->
</option>
```


## 2. 추가구성품<a id="supplement"/>

### Request

```javascript
// param.naverProducts
{
    //다른 속성 생략
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

### Response    

```xml
<product>
    <!-- 다른 Node는 생략 -->
    <supplementSupport>true</supplementSupport> <!-- 추가구성품이 포함되는 경우 필수 설정 -->
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