# KG Mobilians-Mobile Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/mobilians-phone-request-billing-key.md">KO</a>

You can request for a billing key and initial payment together through the KG Mobilians-Mobile payment window. Subsequent payments using the billing key must be for the same amount and within 5 days of the day of the initial payment on monthly basis.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.

## 1. Set up PG

Use the following guide to set up KG Mobilians-Mobile as PG in test mode:
- <a href="https://guide.iamport.kr/1e4b6360-ba3e-4715-ac63-f9dc7bb103d9" target="_blank">KG Mobilians Subscription Test Mode Configuration</a>

## 2. Request billing key

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `mobilians`.
- `customer_uid` : must be specified for billing key registration.
- `amount` : must be specified.
	- Requests both billing key + initial payment.
	- Subsequent payments using the billing key must be for the same amount and within 5 days of the day of the initial payment on monthly basis.

```javascript
IMP.request_pay({
	pg : 'mobilians',
	pay_method : 'phone', // only 'phone' supported.
	merchant_uid : '{Order ID}', // Example: issue_billingkey_monthly_0001
	name : 'Order name: Billing key request test',
	amount : 10000,
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

## 3. Request payment with billing key

After successfully getting the billing key, the billing key is stored on the i'mport server using the specified `customer_uid` as the unique key. For security reasons, the server cannot directly access the billing key. Subsequent payments can be requested by calling the REST API([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) with the `customer_uid` as follows:

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":10000}' \
     https://api.iamport.kr/subscribe/payments/again
```