# NHN KCP General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/kcp.md)

This guide covers only NHN KCP specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up NHN KCP as PG in test mode:
<a href="https://guide.iamport.kr/bc4ded5d-c9f0-4f0e-bd1c-160859748fdd" target="_blank">NHN KCP General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.


- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `kcp`.
- `pay_method` : 
    - card, samsung, trans, vbank, phone, or payco(PAYCO hub)
    - payco: open PAYCO payment window serviced by NHN KCP ([Apply for NHN KCP PAYCO Hub](https://sir.kr/main/service/p_payco_hub.php)) 


```javascript
IMP.request_pay({
    pg : 'kcp',
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

When requesting escrow payment, product-related information must be passed as an additional parameter (`kcpProducts`) for partial delivery purposes.  
`kcpProducts` is an array of objects consisting of the following 4 required properties. The `amount` value has no relation to the payment amount (`param.amount`) value and is not used for comparison.

- `orderNumber` : Product order number
- `name` : Product name
- `quantity` : Quantity
- `amount` : Product price

```javascript
IMP.request_pay({
    pg : 'kcp',
    escrow : true, // Set for escrow payment
    kcpProducts : [
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
    //* ...Omitted (Check sample code in README file)... *//
})
```

