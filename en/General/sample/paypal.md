# PayPal General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/paypal.md)

This guide covers only PayPal specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Redirect method](../README.md#redirect)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up PayPal as PG in test mode:
<a href="https://guide.iamport.kr/458b899c-1058-4055-8b73-a171b5354c2e" target="_blank">PayPal General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, the page is redirected to `m_redirect_url` after calling `IMP.request_pay(param, callback)`.  

ℹ️ Two payment methods are available for Korean businesses: **PayPal Standard** and **Express Checkout**. i'mport applies **Express Checkout** due to its low user bounce rate and no missing payment data errors.

- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `paypal`.
- `pay_method` : only card supported.
- `currency` : only USD supported. Please contact the Customer Service to inquire about other currencies.

```javascript
IMP.request_pay({
    pg : 'paypal',
    pay_method : 'card',
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14.20,
    currency : 'USD',
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    m_redirect_url : '{URL to redirect to after payment approval}' // Example: https://www.my-service.com/payments/complete
});
```

 
