# KICC Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/kicc-request-billing-key.md">KO</a>

You can request for a billing key through the KICC payment window and then request subsequent paymentsayment with the billing key.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.

## 1. Set up PG

Use the following guide to set up KICC as PG in test mode:
- <a href="https://guide.iamport.kr/98b6d582-0dd2-43e0-95bd-037c5e05b416"  target="_blank">KICC Subscription Test Mode Configuration</a>

## 2. Request billing key

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`. In mobile browsers, the page is redirected to  `m_redirect_url`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `kicc`.
    - If you have multiple merchant IDs (each for general and subscription) issued by KICC, set to `kicc.{Merchant ID}`
- `customer_uid` : must be specified for billing key registration.
- `amount` : amount to display in the payment window without processing payment approval. To make the initial payment after getting the billing key, specify the amount to display in the payment window and then [request payment using the billing key](#request-pay).

```javascript
IMP.request_pay({
	pg : 'kicc',
	pay_method : 'card', // only 'card' supported.
	merchant_uid : '{Order ID}', // Example: issue_billingkey_monthly_0001
	name : 'Order name: Billing key request test',
	amount : 0, // For display purpose only (no payment approval).
	customer_uid : '{Unique ID for the card (billing key)}', // Required (Example: gildong_0001_1234)
	buyer_email: "johndoe@gmail.com",
    buyer_name: "John Doe",
	buyer_tel : '02-1234-1234',
	m_redirect_url : '{redirect URL}' // Example: https://www.my-service.com/payments/complete/mobile (for mobile only)
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