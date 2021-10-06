# CHAI General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/chai.md)

This guide covers only CHAI specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Redirect method](../README.md#redirect)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up CHAI as PG in test mode:
<a href="https://guide.iamport.kr/97a21fe1-3e5f-4e77-9de1-38e55ebe73b1" target="_blank">CHAI General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, the page is redirected to `m_redirect_url` after calling `IMP.request_pay(param, callback)`.

- `pg`:
    - If you only have one merchant ID issued by CHAI, set to `chai`.
    - If you have multiple merchant IDs issued by CHAI, set to `chai.{public_key}`.
- `pay_method` : only 'trans' supported.

```javascript
IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans',
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    m_redirect_url : '{URL to redirect to after payment approval}' // Example: https://www.my-service.com/payments/complete
});
```


