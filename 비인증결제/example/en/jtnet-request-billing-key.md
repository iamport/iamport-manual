# JTNet Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/jtnet-request-billing-key.md">KO</a>

You can either request for a billing key or billing key + initial payment through the JTNet payment window and then request subsequent payments with the billing key.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.
<Br />

ℹ️ One-time payments can be made using the key-in payment REST API. For more information, refer to the [JTNet Subscription (Billing) Integration Guide (REST API)](./jtnet-api-billing-key.md).

## 1. Set up PG

Use the **1) Payment window** section of the following guide to set up JTNet as PG in test mode:
- <a href="https://guide.iamport.kr/84ffdcba-3a54-47a7-be49-c04593977e22" target="_blank">JTNet Subscription Test Mode Configuration</a>

## 2. Request billing key (optional initial payment)

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`. In mobile browsers, the page is redirected to  `m_redirect_url`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `jtnet`.
- `customer_uid` : must be specified for billing key registration.
- `amount` : 
	- If only requesting for billing key, set to 0. 
	- If requesting both billing key and initial payment, specify the payment amount.

```javascript
IMP.request_pay({
	pg : 'jtnet',
	pay_method : 'card', // only 'card' supported.
	merchant_uid : '{Order ID}', // Example: issue_billingkey_monthly_0001
	name : 'Order name: Billing key request test',
	amount : 0, // For display purpose only (set actual amount to also request payment approval).
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


## 3. Request payment with billing key

After successfully getting the billing key, the billing key is stored on the i'mport server using the specified `customer_uid` as the unique key. For security reasons, the server cannot directly access the billing key. Subsequent payments can be requested by calling the REST API([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) with the `customer_uid` as follows:

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```
