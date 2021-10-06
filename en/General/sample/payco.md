# PAYCO General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/payco.md)

This guide covers only PAYCO specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Callback method](../README.md#callback)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up PAYCO as PG in test mode:
<a href="https://guide.iamport.kr/09f252f7-f15e-4e1c-ad39-d526486b463b" target="_blank">PAYCO General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `payco`.
- `pay_method` : overwritten by the option selected in PAYCO app.

```javascript
IMP.request_pay({
    pg : 'payco',
    pay_method : 'card', // Can be omitted
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456'
}, function(rsp) { // callback logic
	//* ...Omitted (Check sample code in README file)... *//
});
```  

