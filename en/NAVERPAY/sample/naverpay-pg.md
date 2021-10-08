# Naver Pay (Standard) General Payment Integration Guide

:globe_with_meridians: [KO](/NAVERPAY/sample/naverpay-pg.md)  

You can integrate Naver Pay (Standard) on the payment page as a payment option among other PGs for general payment products.  

This guide covers only Naver Pay specific information. Check the following general payment integration guide.  

- [PC/Mobile web integration](/en/General/README.md#pc-mobile)

## 1. Set up PG

Use the **2) Standard** section of the following guide to set up Naver Pay as PG in test mode:  
<a href="https://guide.iamport.kr/485c6da8-01d7-4900-bc05-76005e5477ba" target="_blank">Naver Pay (Checkout/Standard) Test Mode Configuration</a>  


## 2. Open payment window  

To open the Naver Pay window, call [IMP.request_pay(param)](https://docs.iamport.kr/en-US/tech/imp#request_pay).  

In both PC and mobile, the popup method (invoke callback) or redirection method (redirect to `m_redirect_url`) can be performed by setting the `naverPopupMode` parameter. For more information, refer to [Post-payment processing in PC and mobile](#method).  

- `pg` : 
   - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `naverpay`.
- `buyer_tel` : Required.
- `naverUseCfm` : Expiration date (string in yyyyMMdd format, must be same as the date of payment or later).
	- Specify if contract between Naver Pay and merchant requires this value for the specified product type (such as music or performance tickets).  
- `name`: Order name.
	- If multiple products are defined in the `naverProducts` parameter, Naver Pay automatically appends `and 2 others`. Hence, set this to the name of the first product (`naverProducts[0].name`) in the array.
- `tax_free` : Tax-free amount (out of total `amount`). Default value is 0.
- `naverPopupMode` : Option to proceed via popup window (true/false).
	- If `false`, payment process will proceed via page redirection and you must specify `m_redirect_url`.
- `m_redirect_url` : URL to redirect to after payment approval when using redirection method (`naverPopupMode`: false).  
- [naverProducts](#naverProducts) : Product info (required). This corresponds with the `productItems` parameter mentioned in the Naver Pay manual.

```javascript
IMP.request_pay({
    pg : 'naverpay',
	merchant_uid: '{Merchant created Order ID}', //Example: order_no_0001
    name : 'Order name: Test payment',
    amount : 14000,
    tax_free : 4000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'Customer name',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Samseong-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
	naverUseCfm : '20201001', //Expiration date
    naverPopupMode : true, //Enable popup mode
	m_redirect_url : "{URL to redirect to after payment approval}", //Example: http://yourservice.com/payments/complete
    naverProducts : [{
      "categoryType": "BOOK",
			"categoryId": "GENERAL",
			"uid": "107922211",
			"name": "History of the World",
			"payReferrer": "NAVER_BOOK",
			"count": 10
		},
		{
			"categoryType": "MUSIC",
			"categoryId": "CD",
			"uid": "299911002",
			"name": "BTS",
			"payReferrer": "NAVER_BOOK",
			"count": 1
		}]
}, function(rsp) { //Callback that is invoked in pop-up mode or when an error occurs before starting the payment process.
	if ( rsp.success ) {
    	//[1] Pass imp_uid using jQuery ajax to retrieve payment information from the server
    	jQuery.ajax({
    		url: "/payments/complete", //Be cautious about cross-domain error
    		type: 'POST',
    		dataType: 'json',
    		data: {
	    		imp_uid : rsp.imp_uid
	    		//Add other required data as needed
    		}
    	}).done(function(data) {
    		//[2] Server-side payment amount check via REST API and service processing are successful
    		if ( everythings_fine ) {
    			var msg = 'Payment successful.';
    			msg += '\Payment ID: ' + rsp.imp_uid;
    			msg += '\nOrder ID: ' + rsp.merchant_uid;
    			msg += '\Payment amount: ' + rsp.paid_amount;
    			msg += 'Credit card authorization number: ' + rsp.apply_num;
    			
    			alert(msg);
    		} else {
    			//[3] Payment is not complete.
    			//[4] Payment amount check failed -> payment has been automatically cancelled.
    		}
    	});
    } else {
        var msg = 'Payment failed.';
        msg += 'Error: ' + rsp.error_msg;
        
        alert(msg);
    }
});
```

### `naverProducts` parameter<a id="naverProducts"></a>
 
`naverProducts` is an array of objects consisting of the following 6 properties:  

- categoryType (Required): Refer to [NPay Developers](https://developer.pay.naver.com/docs/v2/api#etc-etc_product)
- categoryId (Required): Refer to [NPay Developers](https://developer.pay.naver.com/docs/v2/api#etc-etc_product)
- uid (Required) : Merchant created product ID in general. Refer to [NPay Developers](https://developer.pay.naver.com/docs/v2/api#etc-etc_product)
- name (Required) : Product name
- count (Required) : Selected quantity
- payReferrer (Optional) : Funnel path to Naver Pay. Enter only when in partnership agreement with other services on the Naver platform. Refer to [NPay Developers](https://developer.pay.naver.com/docs/v2/api#etc-etc_product_ref)

### Post-payment processing in PC and mobile<a id="method"></a>

**PC**  
- To avoid Chrome80 issue (SameSite=Lax by default), payment is made via pop-up window using the payment API or via page redirection.
- Naver Pay NPay Developers specifies that the use of pop-up window is restricted in Internet Explorer due to the loss of the window.opener object, but i'mport supports the pop-up window method in IE.  

**Mobile**  
- Payment is processed by redirection or pop-up window using the new Naver Pay SDK.
- In mobile, there are many restrictions on payment via pop-up window as described below. Hence, it is common to proceed with the page redirection method as recommended by the Naver Pay code review team. If you use page redirection for both PC and mobile, you can implement the same post-payment processing logic in both.
	- Payment via WebView: Increased complexity of Native level implementation (need to create a separate WebView by detecting the `window.open` event).
	- Browsers, such as Safari: Pop-ups forcibly blocked due to user settings.  

## 3. Other  

### Additional properties for Refund API request

i'mport refund request API, `POST /payments/cancel`, requires the following additional properties:

- `extra.requester` : API requester. Select one of:
	- customer: Customer initiated
	- admin (default value): Admin initiated
- `reason`: Reason for refund.

**Example (json)**
```javascript
{
  "imp_uid" : "imp_123412341234", //i'mport transaction ID
  "amount" : 3000, //Amount to refund
  "reason": "reason for refund"
  "extra" : {
    "requester" : "customer"
  }
}
```

**Example (form-urlencoded)**
```
imp_uid=imp_123412341234&amount=3000&extra[requester]=customer
```

