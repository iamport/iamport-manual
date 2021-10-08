# Naver Pay (Standard) Subscription/Recurring Payment Integration Guide

:globe_with_meridians: [KO](/NAVERPAY/sample/naverpay-recurring.md)  

You can integrate Naver Pay (Standard) on the payment page as a payment option among other PGs for subscription/recurring payment products.  


## 1. Set up PG

Use the **2) Standard** section of the following guide to set up Naver Pay as PG in test mode:  
<a href="https://guide.iamport.kr/485c6da8-01d7-4900-bc05-76005e5477ba" target="_blank">Naver Pay (Checkout/Standard) Test Mode Configuration</a>  


## 2. Open subscription/recurring payment registration window  

To open the registration window, call [IMP.request_pay(param)](https://docs.iamport.kr/en-US/tech/imp#request_pay).  

In both PC and mobile, the popup method (invoke callback) or redirection method (redirect to `m_redirect_url`) can be performed by setting the `naverPopupMode` parameter. For more information, refer to [Post-registration processing in PC and mobile](#method).   

- `pg` : 
   - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `naverpay`.
- `customer_uid` : Unique key to identify the customer.
  - Must be specified for subscription/recurring payment registration.
  - After registration, you can use the key to request recurring payment approval.
- `amount` : Actual payment is not approved during the registration process (only used to show the amount that will be requested separately after the registration).  
- `buyer_tel` : Required.
- `naverPopupMode` : Option to proceed via popup window (true/false).
	- If `false`, registration process will proceed via page redirection and you must specify `m_redirect_url`.
- `m_redirect_url` : URL to redirect to after registration when using redirection method (`naverPopupMode`: false).
- `naverProductCode` : Product code managed by the merchant.
  - Used to prevent the same customer from registering duplicate payments for the same product.
  - The default value is a randomly generated value allowing for duplicate payment registration. To avoid this, specify a value.  
  
```javascript
IMP.request_pay({
    pg : 'naverpay',
    customer_uid : 'gildong_0001_1234', //customer ID, required.
    merchant_uid: '{Merchant created Order ID}', //Example: order_monthly_0001
    name : 'Slim plan (montly)',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'Customer name',
    buyer_tel : '010-1234-5678', //Required
    buyer_addr : 'Samseong-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    naverProductCode : 'Recurring payment product code',
    naverPopupMode : true, //Enable popup mode
	  m_redirect_url : "{URL to redirect to after registration}", //Example: http://yourservice.com/payments/complete
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
        //[2] Server-side payment registration check via REST API and service processing are successful
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
### Post-registration processing in PC and mobile<a id="method"></a>

**PC**  
- To avoid Chrome80 issue (SameSite=Lax by default), subscription/recurring payment registration is made via pop-up window using the payment API or via page redirection.
- Naver Pay NPay Developers specifies that the use of pop-up window is restricted in Internet Explorer due to the loss of the window.opener object, but i'mport supports the pop-up window method in IE.  

**Mobile**  
- Payment is processed by redirection or pop-up window using the new Naver Pay SDK.
- In mobile, there are many restrictions on payment through the pop-up window as described below. Hence, it is common to proceed with the page redirection method as recommended by the Naver Pay code review team. If you use page redirection for both PC and mobile, you can implement the same post-payment processing logic in both.
	- Payment via WebView: Increased complexity of Native level implementation (need to create a separate WebView by detecting the `window.open` event).
	- Browsers, such as Safari: Pop-ups forcibly blocked due to user settings.  


## 3. Request payments and schedule future payments   

When subscription/recurring payment registration is completed, you can request the initial payment or schedule future payment by calling the payment approval API using the registered `customer_uid`. For how to implement subscription/recurring payments using the i'mport REST API, refer to the [Subscription Payments Guide](https://docs.iamport.kr/en-US/implementation/subscription).  

ℹ️ To prevent accidental duplicate billings during subscription/recurring payment, you must use a different `merchant_uid` (merchant managed order number) each time you call the API for payment registration and initial/recurring payment approval. You can only use the `merchant_uid` once to call an API.  
 
## 3.1 Request initial/future payments

To request a payment approval, call the REST API [/subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again).  

- `customer_uid` : `customer_uid` specified for subscription/recurring payment registration.  
- `merchant_uid` : Merchant managed order number.
- `amount` : Payment amount (can be different from the amount specified for subscription/recurring payment registration).
- `tax_free` : Tax-free amount (out of total `amount`). Default value is 0.
- `name` : Order name.
- `extra.naverUseCfm` : Expiration date (string in yyyyMMdd format, must be same as the date of payment or later).
	- Specify if contract between Naver Pay and merchant requires this value for the specified product type (such as music or performance tickets).   

**Example (json)**
```javascript
{
  "customer_uid" : "gildong_0001_1234",
  "merchant_uid" : "order_monthly_0002", //Cannot be reused
  "amount" : 10000,
  "name" : "Slim plan (initial payment)",
  "extra" : {
    "naverUseCfm" : "20201001"
  }
}
```

**Example (form-urlencoded)**
```
customer_uid={Unique key to identify the customer}&merchant_uid={Order number}&amount=10000&name=Slim plan (initial payment)&extra[naverUseCfm]=20201001
```

## 3.2 Schedule payments

To schedule payments, call the REST API [/subscribe/payments/schedule](https://api.iamport.kr/#!/subscribe/schedule).

- `customer_uid` : `customer_uid` specified for subscription/recurring payment registration.  
- `schedules` : Payment schedule object array (one or more schedules).
	- `merchant_uid` : Merchant managed order number.
	- `schedule_at` : Scheduled time of payment (UNIX timestamp)
	- `amount` : Payment amount (can be different from the amount specified for subscription/recurring payment registration).
	- `extra.naverUseCfm` : Expiration date (string in yyyyMMdd format, must be same as the date of payment or later).
	  - Specify if contract between Naver Pay and merchant requires this value for the specified product type (such as music or performance tickets).  

**Example (json)**
```javascript
{
  "customer_uid" : "gildong_0001_1234",
  "schedules": [
    {
      "merchant_uid" : "order_monthly_0003", //Cannot be reused
      "schedule_at" : 1519862400,
      "amount" : 10000,
      "extra" : {
        "naverUseCfm" : "20201001"
      }
    }
  ]
}
```

**Example (form-urlencoded)**
```
customer_uid={Unique key to identify the customer}&schedules[0][merchant_uid]={Oder number}&schedules[0][schedule_at]={Scheduled time of payment UNIX timestamp}&schedules[0][amount]=10000&schedules[0][extra][naverUseCfm]=20201001
```
