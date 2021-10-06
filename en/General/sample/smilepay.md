# SmilePay General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/SmilePay.md)

This guide covers only SmilePay specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration > Callback method](../README.md#callback)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up SmilePay as PG in test mode:
<a href="https://guide.iamport.kr/8a2de7ab-db8c-4422-be49-ebaf1e5b64ef" target="_blank">SmilePay General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In both PC and mobile browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.

- `pg`: PG
	- If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `smilepay`.
- `pay_method` : overwritten by the option(card, trans, or point) selected in SmilePay app.

```javascript
IMP.request_pay({
    pg : 'smilepay',
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

## 3. Mobile App WebView integration

### 3.1 iOS  

ℹ️ To call the merchant [app_scheme](../README.md#app_scheme) to return to the merchant app after payment is approved through the Smile Pay app, the `app_scheme` value must be registered in advance through the LGCNS contract manager.

SmilePay supports payment via web or app according to the cookie settings.

- Cookies allowed: web or app
- Cookies denied: app only

When you allow cookies, you can choose between web login or app login as follows: 

![SmilePay payment window when cookies are accepted](./screenshot/smilepay-window.png)  

For web login, enabled state of the **Auto Login** toggle button is not maintained when you return to the page. To keep automatic login enabled, load the page into the webView by calling **WebView.loadHTMLString** with the HTML string of the page and `baseURL` (https://www.mysmilepay.com) as follows:

```java
    if (myPG == "smilepay") {
        if let base = URL(string: "https://www.mysmilepay.com") {
            wv.loadHTMLString("<HTML 내용>", baseURL: base)
        }
    }
```

