# Subscription Payment (Billing) Integration by PG

:globe_with_meridians: <a href="https://github.com/iamport/iamport-manual/blob/master/%EB%B9%84%EC%9D%B8%EC%A6%9D%EA%B2%B0%EC%A0%9C/README.md">KO</a>

To make a payment at a desired time, such as subscriptions, recurring payments or pay-as-you-go billing, the customer's card information must be submitted to the credit card company. Using the card information, the credit card company issues a billing key, a payment encryption key that maps to the card information, to enable subsequent payments.<Br />

Card information can be submitted to the credit card company for billing key issuance using:

- **Payment window method**: submit card information through PG's payment window.
- **REST API method**: submit card information via i'mport REST API.

| ℹ️  **For details on subscription payment integration, such as billing key issuance and payment, refer to the <a href="https://docs.iamport.kr/en-US/implementation/subscription">Subscription Payments</a> page.**|
| :--- |

<Br />

As each PG supports different subscription integration methods and settings, please check the details of each PG below. There are PG companies that support both methods.

### Payment Window Method

- [Danal (Credit Card)](./example/en/danal-card-request-billing-key.md)
- [Danal (Mobile)](./example/en/danal-phone-request-billing-key.md)
- [NHN KCP](./example/en/kcp-request-billing-key.md)
- [KG INICIS](./example/en/inicis-request-billing-key.md)
- [JTNet](./example/en/jtnet-request-billing-key.md)
- [KICC](./example/en/kicc-request-billing-key.md)
- [Payco](./example/en/payco-request-billing-key.md)
- [Kakao Pay](./example/en/kakaopay-request-billing-key.md)
- [CHAI](./example/en/chai-request-billing-key.md)
- [KG Mobilians](./example/en/mobilians-phone-request-billing-key.md)
- [Naver Pay](/NAVERPAY/sample/naverpay-recurring.md)

### REST API Method

- [NICE Payments](./example/en/nice-api-billing-key.md)
- [NHN KCP](./example/en/kcp-api-billing-key.md)
- [KG INICIS](./example/en/inicis-api-billing-key.md)
- [JTNet](./example/en/jtnet-api-billing-key.md)
- [Settlebank](./example/en/settlebank-api-billing-key.md)


