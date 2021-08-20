# Subscription Payment (Billing) Integration by PG

:globe_with_meridians: [KO](../../README.md)

To make a payment at a desired time, such as subscriptions, recurring payments or pay-as-you-go billing, the customer's card information must be submitted to the credit card company. Using the card information, the credit card company issues a billing key, a payment encryption key that maps to the card information, to enable subsequent payments.<Br />

Card information can be submitted to the credit card company for billing key issuance using:

- **Payment window method**: submit card information through PG's payment window.
- **REST API method**: submit card information via i'mport REST API.

| ℹ️  **For details on subscription payment integration, such as billing key issuance and payment, refer to the <a href="https://docs.iamport.kr/en-US/implementation/subscription">Subscription Payments</a> page.**|
| :--- |

<Br />

As each PG supports different subscription integration methods and settings, please check the details of each PG below. There are PG companies that support both methods.

### Payment Window Method

- [Danal (Credit Card)](./danal-card-request-billing-key.md)
- [Danal (Mobile)](./danal-phone-request-billing-key.md)
- [NHN KCP](./kcp-request-billing-key.md)
- [KG INICIS](./inicis-request-billing-key.md)
- [JTNet](./jtnet-request-billing-key.md)
- [KICC](./kicc-request-billing-key.md)
- [Payco](./payco-request-billing-key.md)
- [Kakao Pay](./kakaopay-request-billing-key.md)
- [CHAI](./chai-request-billing-key.md)
- [KG Mobilians](./mobilians-phone-request-billing-key.md)
<!--- - [Naver Pay](/NAVERPAY/sample/naverpay-recurring.md) -->

### REST API Method

- [NICE Payments](./nice-api-billing-key.md)
- [NHN KCP](./kcp-api-billing-key.md)
- [KG INICIS](./inicis-api-billing-key.md)
- [JTNet](./jtnet-api-billing-key.md)
- [Settlebank](./settlebank-api-billing-key.md)


