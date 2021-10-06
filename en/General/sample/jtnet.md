# JTNet General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/jtnet.md)

This guide covers only JTNet specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up JTNet as PG in test mode:
<a href="https://guide.iamport.kr/b1f49d68-272d-40ef-a00c-f444179809e7" target="_blank">JTNet General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg`: 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `jtnet`.
    - If you have multiple merchant IDs (each for general and subscription) issued by JTNet, set to `jtnet.{Merchant ID}`.
- `pay_method`: card, trans, vbank, or phone

```javascript
IMP.request_pay({
    pg : 'jtnet',
    pay_method : 'card',
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    m_redirect_url : '{Mobile only - URL to redirect to after payment approval}' // Example: https://www.my-service.com/payments/complete/mobile
}, function(rsp) { // callback logic
	//* ...Omitted (Check sample code in README file)... *//
});
```



