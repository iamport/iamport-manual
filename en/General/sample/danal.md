# Danal General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/danal.md)

This guide covers only Danal specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Callback method](../README.md#callback)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the **1) General payment** or **2) Mobile micropayments** section of the following guide to set up Danal as PG in test mode:
<a href="https://guide.iamport.kr/9865a7f8-4e20-4fe1-89bd-413beae49982" target="_blank">Danal General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set according to pay_method as follows:
	    - card/trans/vbank : If you have multiple merchant IDs (each for general and subscription) issued by Danal, set to `danal_tpay.{Merchant ID}`.
	    - phone : If you have multiple merchant IDs (each for general and subscription) issued by Danal, set to `danal.{Merchant ID}`.
- `pay_method` : card, trans, vbank, or phone
- `buyer_tel`: required

### 2.1. Credit card or account transfer example

```javascript
IMP.request_pay({
    pg : 'danal_tpay',
    pay_method : '{card or trans}',
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
### 2.2. Virtual bank account example  

- `biz_num`: 10-digit business registration number (required)

```javascript
IMP.request_pay({
    pg : 'danal_tpay',
    pay_method : 'vbank',
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    /* ...Omitted... */
    biz_num : '1234567890'   // Required
}, function(rsp) { // callback logic
	//* ...Omitted (Check sample code in README file)... *//
});
```

