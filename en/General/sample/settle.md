# Settlebank General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/settle.md)

This guide covers only Settlebank specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

⚠️ For PC web, it is recommended to use i'mport JavaScript SDK 1.1.7 or later version to avoid pop-up block issues.

## 1. Set up PG

Use the following guide to set up Settlebank as PG in test mode:
<a href="https://guide.iamport.kr/4c37249c-be5e-45f9-84a2-769577d88736" target="_blank">Settlebank General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg`: 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `settle`.
- `pay_method`: card, trans, vbank, or phone
- `buyer_tel`: required

```javascript
IMP.request_pay({
    pg : 'settle',
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

