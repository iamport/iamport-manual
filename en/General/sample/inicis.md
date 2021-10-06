# KG INICIS General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/inicis.md)

This guide covers only KG INICIS specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up KG INICIS as PG in test mode:
<a href="https://guide.iamport.kr/8f617f32-564d-464e-9850-bf7648c609b0" target="_blank">KG INICIS General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `html5_inicis` or `inicis`(for ActiveX).
    - If you have multiple merchant IDs(each for general and subscription) issued by KG INCIS, set to `html5_inicis.{Merchant ID}` or `inicis.{Merchant ID}`(for ActiveX).
- `pay_method` : card, trans, vbank, or phone
- `buyer_tel`: required

```javascript
IMP.request_pay({
    pg : 'html5_inicis',
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

## 3. Mobile App WebView integration

### 3.1 iOS

After an instant account transfer payment is approved, you must make a final payment request from the webView as follows:

```swift
// url switch
open func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    let myAppScheme = "Merchant's appscheme"
    if (url.absoluteString.hasPrefix(myAppScheme)) {
        processInicisTrans(appScheme, url)
        return
    }
    .. Omitted
}

// Send url to webView via POST request
private func processInicisTrans(_ appScheme: String, _ url: URL) {
    func isParseUrl(_ str: String) -> Bool {
        if URL(string: str) != nil {
            return true
        }

        return false
    }

    var scheme = "\(appScheme)?"
    if (!appScheme.contains("://")) {
        scheme = "\(appScheme)://?"
    }

    let m_redirect_url = "m_redirect_url value"

    let removeAppScheme = url.absoluteString.replacingOccurrences(of: scheme, with: "")
    let separated = removeAppScheme.components(separatedBy: "=")
    let redirectUrl = separated.map { s -> String in
        s.removingPercentEncoding ?? s
    }.filter { s in
        s.contains(m_redirect_url)
    }.first

    if let urlStr = redirectUrl, let url = URL(string: urlStr) {
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        webView?.load(request)
    }
}
```

