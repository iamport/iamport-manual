# CHAI Subscription (Billing) Integration Guide `Payment Window`

:globe_with_meridians: [KO](../chai-request-billing-key.md)

You can request for a billing key through the simple payment window of CHAI and then request subsequent paymentsayment with the billing key.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > Payment window](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-b) section of the Subscription Payments guide.

## 1. Set up PG

Use the following guide to set up CHAI as PG in test mode:
- <a href="https://guide.iamport.kr/195ae9aa-d862-4fb6-a637-3065c3dae1e7" target="_blank">CHAI Subscription Test Mode Configuration</a>

## 2. Request billing key

To open the payment window for billing key request, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/tech/imp#request_pay).

In both PC and mobile browsers, the page is redirected to `m_redirect_url` after calling `IMP.request_pay(param, callback)`.

- `pg`:
    - If you only have one merchant ID issued by CHAI, set to `chai`.
    - If you have multiple merchant IDs issued by CHAI, set to `chai.{public_key}`.
- `customer_uid`: must be specified for billing key registration.
- `amount` : amount to display in the payment window without processing payment approval. To make the initial payment after getting the billing key, specify the amount to display in the payment window and then [request payment using the billing key](#request-pay).

```javascript
IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans', // Only supports trans (bank transfer service)
    merchant_uid : '{Order ID}', // Example: issue_billingkey_monthly_0001
    name : 'Order name: Billing key request test',
    amount : 0, // For display purpose only (no payment approval)
    customer_uid : '{Unique ID for the card (billing key)}', // Required (Example: gildong_0001_1234)
    buyer_email: "johndoe@gmail.com",
    buyer_name: "John Doe",
    buyer_tel: "010-4242-4242",
    buyer_addr: "Shinsa-dong, Gangnam-gu, Seoul",
    buyer_postcode: "01181" 
    m_redirect_url : '{redirect URL}' // Example: https://www.my-service.com/payments/complete/mobile
}, function(rsp) {
    if ( !rsp.success ) {
    	// Error occurred before page is redirected
        var msg = 'Unable to process request due to an error.';
        msg += 'Error message : ' + rsp.error_msg;

        alert(msg);
    }
});
```

<a name="request-pay" />

## 3. Request payment with billing key

After successfully getting the billing key, the billing key is stored on the i'mport server using the specified `customer_uid` as the unique key. For security reasons, the server cannot directly access the billing key. Subsequent payments can be requested by calling the REST API([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) with the `customer_uid` as follows:

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"merchant_xxxxxxxx", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```
