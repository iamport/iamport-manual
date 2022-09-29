# PAYMENTWALL Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: [KO](/비인증결제/example/paymentwall-request-billing-key.md)

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.

ℹ️ Merchants can be separately contracted with PAYMENTWALL for integrating subscription payment (billing) using REST API. For more information, refer to the [PAYMENTWALL Subscription (Billing) Integration Guide (REST API)](./paymentwall-api-billing-key.md).


## 1. Set up PG

Use the **1) Payment window** section of the following guide to set up PAYMENTWALL as PG in test mode:

- <a href="https://chaifinance.notion.site/5aa0546574c94f159a7f040585af7359" target="_blank"> PAYMENTWALL Subscription Test Mode Configuration</a>


## 2. Request billing key

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg` :
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set.
    - If there are multiple PG settings, set to `paymentwall`.
    - If you have multiple merchant IDs(each for general and subscription) issued by PAYMENTWALL, set to `paymentwall.{Merchant ID}`.
- `customer_uid` : must be specified for billing key registration.
- `amount` : must be specified.
    - Requests both billing key + initial payment.

```javascript
IMP.request_pay({
	pg : "paymentwall.{Merchant ID}",
	pay_method : 'card', // only 'card' supported.
	merchant_uid : '{Merchant created Order ID}', // Example: issue_billingkey_monthly_0001
	name : 'Order name: Billing key request test',
	amount : 50,
	buyer_email: "iamport@example.com", // Required
        buyer_name: "buyer name",
	buyer_tel : '02-1234-1234',
	buyer_adrr : 'Gangnam-gu, Seoul, Republic of Korea',
        buyer_postcode : '123-456',
        currency : 'USD',
        language : 'en',
	m_redirect_url : '{redirect URL}', // Example: https://www.my-service.com/payments/complete/mobile (for mobile only),
	customer_uid : '{Unique ID for the card (billing key)}', // Required (Example: gildong_0001_1234)
	bypass: {
	      widget_code: "t3_1",  // For Terminal 3, set the corresponding parameters and activate the Default payment window if not set.
	    }
}, function(rsp) {
	if ( rsp.success ) {
		alert('Success');
	} else {
		alert('Failed');
	}
});
```

<a name="request-pay" />


## 3. Request payment with billing key

After successfully getting the billing key, the billing key is stored on the i'mport server using the specified `customer_uid` as the unique key. For security reasons, the server cannot directly access the billing key. Subsequent payments can be requested by calling the REST API([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) with the `customer_uid` as follows:

```
curl -H "Content-Type: application/json" \
-X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
https://api.iamport.kr/subscribe/payments/again
```
