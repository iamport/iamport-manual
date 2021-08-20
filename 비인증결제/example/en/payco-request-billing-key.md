# Payco Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/payco-request-billing-key.md">KO</a>

You can request for a billing key through the Payco simple payment window and then request subsequent paymentsayment with the billing key.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.

## 1. Set up PG

Use the following guide to set up Payco as PG in test mode:
- <a href="https://guide.iamport.kr/d6601ff5-49ec-45a6-8bfb-79752e700942" target="_blank">Payco Subscription Test Mode Configuration</a>

## 2. Request billing key

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `payco`.
- `customer_uid` : must be specified for billing key registration.
- `amount` : amount to display in the payment window without processing payment approval. To make the initial payment after getting the billing key, specify the amount to display in the payment window and then [request payment using the billing key](#request-pay).

```javascript
IMP.request_pay({
	pg : 'payco',
	pay_method : 'card', // only 'card' supported.
	merchant_uid : '{Order ID}', // Example: issue_billingkey_monthly_0001
	name : 'Order name: Billing key request test',
	amount : 0, // For display purpose only (no payment approval).
	customer_uid : '{Unique ID for the card (billing key)}', // Required (Example: gildong_0001_1234)
	buyer_email: "johndoe@gmail.com",
    buyer_name: "John Doe",
	buyer_tel : '02-1234-1234'
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