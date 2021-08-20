# KG INICIS Subscription (Billing) Integration Guide `REST API`

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/example/inicis-api-billing-key.md">KO</a>

Merchants can be separately contracted with KG INICIS for integrating subscription payment (billing) using key-in payment ([POST /subscribe/payments/onetime](https://api.iamport.kr/#!/subscribe/onetime)) and billing key request ([POST /subscribe/customers/{customer_uid}](https://api.iamport.kr/#!/subscribe.customer/customer_save)) REST APIs.<Br />

ℹ️ For more information, refer to the [Register card and get billing key > REST API](https://docs.iamport.kr/en-US/implementation/subscription#issue-billing-a) section of the Subscription Payments guide.

ℹ️ For information about integrating subscription payment (billing) using payment window, refer to the [KG INICIS Subscription (Billing) Integration Guide (Payment Window)](./inicis-request-billing-key.md).

## 1. Set up PG

Use the **2) REST API** section of the following guide to set up KG INICIS as PG in test mode:
- <a href="https://guide.iamport.kr/005331e6-be5d-4cc1-a547-9e78416b77e9" target="_blank">KG INICIS Subscription Test Mode Configuration</a>

## 2. Request one-time payment

To request a one-time payment, use the key-in REST API [POST /subscribe/payments/onetime](https://api.iamport.kr/#!/subscribe/onetime). The card information is not saved during this process.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"merchant_uid":"order_id_8237352", "card_number":"1234-1234-1234-1234", "expiry":"2019-01", "birth":"123456", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/onetime
```      
## 3. Request billing key

To request a billing key, use the billing key request REST API [POST /subscribe/customers/{customer_uid}](https://api.iamport.kr/#!/subscribe.customer/customer_save).

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"card_number":"1234-1234-1234-1234", "expiry":"2025-12", "birth":"820213", "pwd_2digit":"00"}' \
     https://api.iamport.kr/subscribe/customers/your-customer-unique-id
```

## 4. Request billing key + initial payment  

To request a billing key and initial payment, use the key-in REST API [POST /subscribe/payments/onetime](https://api.iamport.kr/#!/subscribe/onetime) with the following parameter specified.

- `customer_uid` : Unique ID for the card (billing key)

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "card_number":"1234-1234-1234-1234", "expiry":"2019-01", "birth":"123456", "amount":3000, "pwd_2digit":"00"}' \
     https://api.iamport.kr/subscribe/payments/onetime
```

## 5. Request payment with billing key

After successfully getting the billing key, the billing key is stored on the i'mport server using the specified `customer_uid` as the unique key. For security reasons, the server cannot directly access the billing key. Subsequent payments can be requested by calling the REST API([POST /subscribe/payments/again](https://api.iamport.kr/#!/subscribe/again)) with the `customer_uid` as follows:

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```