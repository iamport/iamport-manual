# Kakao Pay Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/kakaopay-request-billing-key.md">KO</a>

You can either request for a billing key or billing key + initial payment through the Kakao Pay payment window and then request subsequent payments with the billing key.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.

## 1. Set up PG

Use the following guide to set up Kakao Pay as PG in test mode:
- <a href="https://guide.iamport.kr/0f2bec60-0a7b-4626-9d72-7af82740ceb2" target="_blank">Kakao Pay Subscription Test Mode Configuration</a>

## 2. Request billing key (optional initial payment)

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `kakaopay`.
- `customer_uid` : must be specified for billing key registration.
- `amount` : 
	- If only requesting for billing key, set to 0. 
	- If requesting both billing key and initial payment, specify the payment amount
- pay_method : overwritten by the option(credit card or Kakao money) selected in Kakao Pay app, and the selection is applied for subsequent payments.

```javascript
IMP.request_pay({
	pg : 'kakaopay',
	pay_method : 'card', // can be omitted, ignored if set.
	merchant_uid : '{Order ID}', // Example: issue_billingkey_monthly_0001
	name : 'Order name: Billing key request test',
	amount : 0, // For display purpose only (set actual amount to also request payment approval).
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

ℹ️ Kakao Pay has a limit on the maximum number of payments made with the same billing key per month.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```

