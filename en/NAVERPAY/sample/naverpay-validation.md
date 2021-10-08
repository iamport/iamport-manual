# Request/Response for Naver Pay Product Options and Add-ons

:globe_with_meridians: [KO](/NAVERPAY/sample/naverpay-validation.md)  

- [1. Product options](#product-option)
   - [1.1 Single option product](#product-option1)
   - [1.2 Single option product (option with price difference)](#product-option1-var)
   - [1.3 Multi-option product](#product-option2)
- [2. Add-ons](#supplement)
  
**Request** represents the parameters passed when calling IMP.request_pay(param) to request for payment.    
**Response** represents the response in XML format for the product verification request from Naver Pay and *includes all available options, add-ons and option combinations*.

## 1. Product options<a id="product-option"/>
 
## 1.1 Single option product<a id="product-option1"/>

### Request

- You cannot specify price difference for the option.  

```javascript
// param.naverProducts.Product.Option
{
   //... Other properties omitted ...
	selections : [
		{
			code : "180",
			label : "Size",
			value : "180"
		}
	]
}
```

### Response  

```xml
<optionSupport>true</optionSupport> <!-- Required for option selected products -->
<option>
     <optionItem>
        <type>SELECT</type>
        <name>Size</name>
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


## 1.2 Single option product (option with price difference)<a id="product-option1-var"/>

### Request  

- To specify price difference for the option, set the `optionPrice` and `selectionCode` properties.  

```javascript
// param.naverProducts.Product.Option
{
	optionPrice : 200, //Price difference for the option
	selectionCode : "Large", //Option/price difference combination (corresponds to combination/manageCode of the response)
	selections : [
		{
			code : "180",
			label : "Size",
			value : "180"
		}
	]
}
```

### Response 

```xml
<optionSupport>true</optionSupport> <!-- Required for option selected products -->
<option>
	<!-- ... Other optionItems omitted ... -->
	<combination> <!-- Option/price difference combination -->
		<manageCode>Large</manageCode> <!-- Corresponds to selectionCode of the request -->
		<price>200</price> <!-- Price difference -->
		<options>
			<name>Size</name>
			<id>180</id>
		</options>
	</combination>
   <!-- ... Omitted (combination nodes for all options with price difference) ... -->
</option>
```

## 1.3 Multi-option product<a id="product-option2"/>

### Request  

```javascript
// param.naverProducts.Product.Option
{
   optionPrice : 200, //Price difference for the option
	selectionCode : "R_L", //Option combination (corresponds to combination/manageCode of the response)
	selections : [
		{
			code : "RED",
			label : "Color",
			value : "Red"
		},
		{
			code : "180",
			label : "Size",
			value : "180"
		}
	]
}
```

### Response  

```xml
<optionSupport>true</optionSupport> <!-- Required for option selected products -->
<option>
	<optionItem> <!-- Option 1 -->
      <type>SELECT</type>
      <name>Color</name>
      <value>
         <id>RED</id>
         <text>Red</text>
      </value>
      <value>
         <id>BLUE</id>
         <text>Blue</text>
      </value>
   </optionItem> <!-- Option 2 -->
   <optionItem>
      <type>SELECT</type>
      <name>Size</name>
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
            
   <!-- Specify all possible combinations for multiple option types (optionItems) -->
   <combination>
      <manageCode>R_L</manageCode> <!-- Corresponds to selectionCode of the request -->
      <price>200</price> <!-- Price difference -->
      <options>
         <name>Color</name>
         <id>RED</id>
      </options>
      <options>
         <name>Size</name>
         <id>180</id>
      </options>
   </combination>
   <!-- ... Other option combinations omitted ... -->
</option>
```


## 2. Add-ons<a id="supplement"/>

### Request

```javascript
// param.naverProducts
{
    //... Other properties Omitted ...
    supplements : [ 
        {
            id : "supplement-a",
            name : "Add-on A",
            price : 1000, //Price per item
            quantity : 1
        },
        {
            id : "supplement-b",
            name : "Add-on B",
            price : 1200, //Price per item
            quantity : 2
        }
    ]
}
```

### Response  

```xml
<product>
    <!-- ... Other nodes omitted ... -->
    <supplementSupport>true</supplementSupport> <!-- Required for products with add-ons -->
    <supplement>
        <id>supplement-a</id>
        <name>Add-on A</name>
        <price>1000</price> <!-- Price per item -->
        <stockQuantity>24</stockQuantity> <!-- Available quantity -->
        <status>true</status> <!-- Whether add-on is available for purchase -->
    </supplement>
    <supplement>
        <id>supplement-b</id>
        <name>Add-on B</name>
        <price>1200</price> <!-- Price per item -->
        <stockQuantity>24</stockQuantity> <!-- Available quantity -->
        <status>true</status> <!-- Whether add-on is available for purchase -->
    </supplement>
</product>
```