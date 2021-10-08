# Naver Pay Shipping Policy and Examples

:globe_with_meridians: [KO](/NAVERPAY/sample/naverpay-shipping.md)  

Naver Pay supports the following shipping methods.  

- [1. Free shipping](#free)
- [2. Fixed rate](#fixed)
- [3. Conditional Free shipping](#conditional)
- [4. Quantity-based shipping](#quantity-based)
- [5. Extended area surcharge](#location-based)
  
## `naverProducts.shipping` property  

The JSON representation of `naverProducts.shipping` property when calling [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay) is as follows:  

```javascript
{
	groupId : "Shipping group ID",
	method : "DELIVERY", //DELIVERY (Courier Delivery/Parcel/Registered), QUICK_SVC (Quick Service), DIRECT_DELIVERY (Direct Delivery), VISIT_RECEIPT (Private Pick-up), NOTHING (No Delivery)
	baseFee : 400, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : {
		freeByThreshold : 20000, //Conditional free shipping threshold
		repeatByQty : 4, //Quantity-based shipping (Incremental shipping charge per quantity)
		rangesByQty : [ //Quantity-based shipping (Additional shipping by custom quantity range)
			{from: 10, surcharge:4000},
			{from: 20, surcharge:6000}
		],
		surchargesByArea : [ //Extended area surcharge
			{area:"island", surcharge:2000}, //Mountainous and island area
			{area:"jeju", surcharge:3000} //Jeju
		]
		//	surchargesByArea : "API" //Determine additional fee via API (Delete other extended area surcharge settings when using this setting)
	}
}
```

## 1. Free shipping<a id="free"/>

To apply free shipping, specify as follows (without `feeRule` and `feePayType`):  

```javascript
{
	//* ...Omitted... *//
	baseFee : 0 //Base shipping rate
}
```

## 2. Fixed rate<a id="fixed"/>  

To apply fixed shipping rate, specify as follows (without `feeRule`):  

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED" //PREPAYED or CASH_ON_DELIVERY
}
```

## 3. Conditional free shipping<a id="conditional"/>

To waive the shipping charges when the total amount of your order is greater than or equal to the specified threshold value, specify as follows:  

In the following example, the base shipping rate is 2,500 won, and shipping is free when the total amount of your order is 20,000 won or more.  

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : { //Shipping policy
		freeByThreshold : 20000 //Free shipping threshold
	}
}
```

## 4. Quantity-based shipping<a id="quantity-based"/>  

You can apply additional shipping fee for multiple oversized or heavy items.  

## 4.1 Incremental shipping charge per quantity 

You can increment shipping charges per item quantity based on the base shipping rate.  

In the following example, the base shipping rate is 2,500 won, and a shipping charge of 2,500 won is added for every 10 items.  

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : { //Shipping policy
		repeatByQty : 10 //Quantity per which shipping fee is added
	}
}
```

## 4.2 Quantity-based shipping fee by custom quantity range  
 
You can apply additional shipping fee for up to 3 custom quantity ranges.  

The example below applies the following total shipping charges based on item quantity.  

- 1-10 items: Base rate = 2,500 won total
- 11-20 items: Base rate + 2,5000 = 5,000 won total
- 21 or more items: Base rate + 7,5000 = 10,000 won total  

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : { //Shipping policy
		rangesByQty : [ // Quantity-based shipping (Additional shipping by custom quantity range)
			{from: 11, surcharge:2,500}, //Add 2,500 won for 11-20 items 
			{from: 21, surcharge:7,500} //Add 7,500 won for 21 or more items
		]
	}
}
```


## 5. Extended area surcharge<a id="location-based"/>

You can add shipping surcharge for extended areas.

## 5.1 Surcharge calculated by Naver Pay  

You can configure Naver Pay to automatically calculate shipping surcharge for up to 3 areas based on the postal code.

1. Base inland shipping rate: `baseFee`
2. Mountainous and island area (Geoje-do, Ulleungdo, Ganghwa-do, etc.): *Set `area` to "island".*
3. Jeju Island: *Set `area` to "jeju".* If there is no area definition for Jeju, Jeju Island is considered a mountainous and island area.

### Mountainous and island area (including Jeju) surcharge  

In the following example, the base inland shipping rate is 2,500 won, and all mountainous and island areas including Jeju Island are charged an additional 2,000 won, for a total of 4,500 won.   

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : {
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000} // Mountainous and island area including Jeju surcharge
		]
	}
}
```

### Mountainous and island area (excluding Jeju) surcharge  

In the example below, the base inland shipping rate is 2,500 won, and the total shipping charges are as follows:

- Mountainous and island area: Base rate + 2,000 = 4,500 won total  
- Jeju: Base rate + 4,500 = 7,000 won total  

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : {
		surchargesByArea : [ //array or string(API)
			{area:"island", surcharge:2000}, //Mountainous and island area surcharge
			{area:"jeju", surcharge:4500} //Jeju area surcharge
		]
	}
}
```

## 5.2 Custom local shipping rate

If you need to set local shipping rates by city, county, or district for services like flower delivery or quick service, you can use the local shipping rate request API to customize the shipping rate.  

In the following example, Naver Pay calls the API to request the local shipping rate from the merchant. To implement this, the merchant needs to store shipping rates by zip codes in the database.  

```javascript
{
	//* ...Omitted... *//
	baseFee : 2,500, //Base shipping rate
	feePayType : "PREPAYED", //PREPAYED or CASH_ON_DELIVERY
	feeRule : {
		surchargesByArea : "API" //Surcharge calculated via API
	}
}
```

### Request/response for local shipping rate

Naver Pay receives the local shipping rate information in XML format by sending an HTTP request for the shipping address to the `Mountainous and Island area surcharge request URL`, which was specified at Naver Pay sign-up, from the checkout page.  

**HTTP request example**

- productId[N]: Product ID
- zipcode: Zip code of the shipping address
- address1: Shipping address (base64 URL encoded value)

```
http://{Mountainous and island area surcharge request URL}?productId[0]=XXX&productId[1]=XXX&zipcode=463050&address1=6rK96riw64-EIOyEseuCqOyLnCDrtoTri7nqtawg7ISc7ZiE64-Z
```

**XML response example (Content-Type: application/xml)**

```xml
<additionalFees> <!-- Add a node for each productId passed via request's query parameter -->
   <additionalFee> <!-- Additional charge for each product  -->
          <id>P001</id> <!-- productId -->
          <surprice>0</surprice> <!-- Additional fee -->
   </additionalFee>
   <additionalFee>
          <id>P002</id>
          <surprice>1000</surprice>
   </additionalFee>
<additionalFees>
```

