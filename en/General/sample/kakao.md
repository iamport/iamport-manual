# Kakao Pay General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/kakao.md)

This guide covers only Kakao Pay specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Callback method](../README.md#callback)
- [Mobile app WebView integration](../README.md#webview)

> As of May 2018, Kakao Pay Corp., not LGCNS, will provide Kakao Pay payment services. To allow time for conversion, i'mport will support both providers.
>
> Use the same code with the following pg\_provider parameter values to differentiate between the two providers.
>
> Old Kakao Pay - pg\_provider : kakao  
> New Kakao Pay - pg\_provider : kakaopay

## 1. Set up PG

Use the following guide to set up Kakao Pay as PG in test mode:
<a href="https://guide.iamport.kr/51110e50-4c48-43bc-b0b5-a2161f4b5af8" target="_blank">Kakao Pay General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `kakaopay`.
- `pay_method` : overwritten by the option(credit card or Kakao money) selected in Kakao Pay app.


```javascript
IMP.request_pay({
    pg : 'kakaopay',
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

