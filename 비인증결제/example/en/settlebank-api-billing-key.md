# Settlebank Subscription (Billing) Integration Guide `REST API`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/settlebank-api-billing-key.md">KO</a>

Settlebank supports key-in payment REST API ([POST /subscribe/payments/onetime](https://api.iamport.kr/#!/subscribe/onetime)) for one-time payment or billing key + initial payment request. You cannot use the REST API `/subscribe/customers/{customer_uid}` to request for a billing key only. Settlebank does not support payment window method for subscription payment integration.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > REST API](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-a) section of the Subscription Payments guide.

## 1. Set up PG

Use the following guide to set up Settlebank as PG in test mode:
- <a href="https://guide.iamport.kr/dc47a51b-37f9-4662-a94f-95a481236204" target="_blank">Settlebank Subscription Test Mode Configuration</a>


## 2. Request one-time payment

To request a one-time payment, use the key-in REST API [POST /subscribe/payments/onetime](https://api.iamport.kr/#!/subscribe/onetime). The card information is not saved during this process.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"merchant_uid":"order_id_8237352", "card_number":"1234-1234-1234-1234", "expiry":"2019-01", "birth":"123456", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/onetime
```


## 3. Request billing key + initial payment  

To request a billing key and initial payment, use the key-in REST API [POST /subscribe/payments/onetime](https://api.iamport.kr/#!/subscribe/onetime) with the following parameters specified.

- `customer_uid` : Unique ID for the card (billing key).
- `birth` : 
     - If 6-digit birth date requirement is specified in the Settlebank contract, set this parameter.
     - Otherwise, set to a dummy value (*123456*).

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "card_number":"1234-1234-1234-1234", "expiry":"2019-01", "birth":"123456", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/onetime
```


## 4. Request payment with billing key

After successfully getting the billing key, the billing key is stored on the i'mport server using the specified `customer_uid` as the unique key. For security reasons, the server cannot directly access the billing key. Subsequent payments can be requested by calling the REST API([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) with the `customer_uid` as follows:

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
