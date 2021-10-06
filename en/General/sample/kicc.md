# KICC General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/kicc.md)

This guide covers only KICC specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up KICC as PG in test mode:
<a href="https://guide.iamport.kr/818e4c81-a14f-42c8-8515-210d4299f3b1" target="_blank">KICC General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `kicc`.
    - If you have multiple merchant IDs (each for general and subscription) issued by KICC, set to `kicc.{Merchant ID}`.
- `buyer_tel`: required

```javascript
IMP.request_pay({
    pg : 'kicc',
    pay_method : 'card',
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    m_redirect_url : '{Mobile only - URL to redirect to after payment approval}' // Example: https://www.my-service.com/payments/complete/mobile
}, function(rsp) { // callback logic
	//* ...Omitted (Check sample code in README file)... *//
});
```

### Escrow payment

KICC supports escrow payments only for cash payment methods (instant account transfer or virtual bank account).

When making an escrow payment, you must enter the following required parameters:

 - `buyer_name` : Customer name
 - `buyer_email` : Customer email
 - `buyer_tel` : Customer phone number
 - `kiccProducts` : An array of objects consisting of the following 4 required properties. The `amount` value has no relation to the payment amount (`param.amount`) value and is not used for comparison.
    - `orderNumber` : Product order number
    - `name` : Product name
    - `quantity` : Quantity
    - `amount` : Product price

```javascript
IMP.request_pay({
    pg : 'kicc',
    escrow : true, // Set for escrow payment
    pay_method : 'vbank', //trans or vbank
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    kiccProducts : [
    	{
			"orderNumber" : "xxxx",
			"name" : "Product A",
			"quantity" : 3,
			"amount" : 1000
		},
		{
			"orderNumber" : "yyyy",
			"name" : "Product B",
			"quantity" : 2,
			"amount" : 3000
		}
	],
    m_redirect_url : '{Mobile only - URL to redirect to after payment approval}' // Example: https://www.my-service.com/payments/complete/mobile
}, function(rsp) { // callback logic
	//* ...Omitted (Check sample code in README file)... *//
});
```

  
