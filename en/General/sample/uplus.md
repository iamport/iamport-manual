# Toss Payments General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/uplus.md)

This guide covers only Toss Payments specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up Toss Payments as PG in test mode:
<a href="https://guide.iamport.kr/01cd8f06-6952-493c-8b77-9c4f235c464c" target="_blank">Toss Payments General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg`: 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `uplus`.  
    *Toss Payments, previously LG Uplus's PG division, maintains the existing `pg` value of `uplus`.*
- `pay_method`: card, trans, vbank, or phone

```javascript
IMP.request_pay({
    pg : 'uplus',
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
 