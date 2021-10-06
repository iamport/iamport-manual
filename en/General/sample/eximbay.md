# Eximbay General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/eximbay.md)

This guide covers only Eximbay specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Callback method](../README.md#callback)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up Eximbay as PG in test mode:
<a href="https://guide.iamport.kr/cdf5c3aa-2529-40ed-bc5a-9daba183c655" target="_blank">Eximbay General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg`: PG
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `eximbay`.
- `pay_method`: Payment method
	- wechat   
	- alipay   
	- card (foreign credit card)
	- unionpay   
	- tenpay
	- econtext (payment via Japanese convenience store)
- `language`: Language
	- ko: Korean    
	- en: English  
	- zh: Chinese    
	- jp: Japanese
- `currency`: Currency (all currencies supported by Eximbay)      
	- KRW      
	- USD      
	- EUR      
	- GBP      
	- JPY      
	- THB      
	- SGD      
	- RUB      
	- HKD      
	- CAD      
	- AUD
- `buyer_tel`: Required

```javascript
IMP.request_pay({
    pg : 'eximbay',
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
 
