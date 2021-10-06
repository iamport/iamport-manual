# Alipay General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/alipay.md)

This guide covers only Alipay specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Redirect method](../README.md#redirect)
- [Mobile app WebView integration](../README.md#webview)

> To provide Alipay payment service, you must be contracted with **Nice Payments**, a Korean e-payment service provider.  
> Basic currency is USD. Adding KWR support requires a prior agreement with Nice Payments.

## 1. Set up PG

Use the following guide to set up Alipay as PG in test mode:
<a href="https://guide.iamport.kr/f73c2a59-2fc3-4ea5-9765-6ff341e48916" target="_blank">Alipay General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, the page is redirected to `m_redirect_url` after calling `IMP.request_pay(param, callback)`.

- `pg` : 
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `alipay`.
- `pay_method` : overwritten by the option selected in Alipay payment window.
- `currency` : `USD` (default value). (USD/KRW supported)

```javascript
IMP.request_pay({
    pg : 'alipay',
    pay_method : 'card', // can be omitted, ignored if set.
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 1.2, // $1.2 in USD
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    m_redirect_url : '{URL to redirect to after payment approval}' // Example: https://www.my-service.com/payments/complete
});
```


