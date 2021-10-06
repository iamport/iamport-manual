# Blue Walnut General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/bluewalnut.md)

This guide covers only Blue Walnut specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Callback method](../README.md#callback)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up Blue Walnut as PG in test mode:
<a href="https://guide.iamport.kr/642be4d3-47e3-45c8-9454-b4272247c6b5" target="_blank">Blue Walnut General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `bluewalnut`.
	- If you have multiple merchant IDs(each for general and subscription) issued by Blue Walnut, set to `bluewalnut.{Merchant ID}`.
- `pay_method` : card, trans, vbank, or phone


```javascript
IMP.request_pay({
    pg : 'bluewalnut',
    pay_method : 'card',
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
