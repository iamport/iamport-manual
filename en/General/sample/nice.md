# NICE Payments General Payment Integration Guide

:globe_with_meridians: [KO](/인증결제/sample/nice.md)

This guide covers only NICE Payments specific information. Check the following general payment integration sections in the README file.

- [PC/Mobile web integration](../README.md)
- [Mobile app WebView integration](../README.md#webview)

## 1. Set up PG

Use the following guide to set up NICE Payments as PG in test mode:
<a href="https://guide.iamport.kr/94861fdf-c937-473b-81f8-eba6bb4cff9f" target="_blank">NICE Payments General Test Mode Configuration</a>

## 2. Request payment

To open the payment window, call [IMP.request_pay(param, callback)](https://docs.iamport.kr/en-US/sdk/javascript-sdk#request_pay).  

In PC browsers, callback is invoked after calling `IMP.request_pay(param, callback)`.  In mobile browsers, the page is redirected to `m_redirect_url`.

- `pg` : 
    - If not specified and this is the only PG setting that exists, `default PG` is automatically set. 
	- If there are multiple PG settings, set to `nice`.
- `pay_method` : card, trans, vbank, or phone
- `niceMobileV2` : Option to apply the new version of Nice Mobile (default: false). This is provided for backwards compatibility.

```javascript
IMP.request_pay({
    pg : 'nice',
    pay_method : 'card',
    merchant_uid : '{Merchant created Order ID}', // Example: order_no_0001
    name : 'Order name: Test payment request',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : 'John Doe',
    buyer_tel : '010-1234-5678',
    buyer_addr : 'Shinsa-dong, Gangnam-gu, Seoul',
    buyer_postcode : '123-456',
    m_redirect_url : '{Mobile only - URL to redirect to after payment approval}', // Example: https://www.my-service.com/payments/complete/mobile
    niceMobileV2 : true // Set to apply new mobile version
}, function(rsp) { // callback logic
	//* ...Omitted (Check sample code in README file)... *//
});
```

## 3. Mobile App WebView integration

### 3.1 iOS

After instant account transfer (BankPay) payment is approved, the final payment request must be made from webView with the data received through `AppDelegate` or `SceneDelegate` as follows:

```swift
// AppDelegate.swift (when using SwiftUI)
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
				Iamport.shared.processBankPayPayment(url)
        return true
    }


// SceneDelegate.swift (when using SwiftUI)
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        if let url = URLContexts.first?.url {
            Iamport.shared.processBankPayPayment(url)
        }
    }
```

```swift
var bankTid: String?
var niceTransUrl: String?

// url switch
open func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
	if(uri.scheme == "kftc-bankpay") {
	    let queryParams = url.queryParams()
	    bankTid = (queryParams["user_key"] as? String)
	    niceTransUrl = (queryParams["callbackparam1"] as? String)
	
	    // Omitted.. Open BankPay app 
	}
}

// Post-processing after receiving result from BankPay app
public func processBankPayPayment(_ url: URL) {
    
    // Retrieve bankpaycode and bankpayvalue and pass them as bankpay_code and bankpay_value, respectively
    guard let queryItems = URLComponents(string: url.absoluteString)?.queryItems,
            let bankpayCode = (queryItems.filter({ $0.name == "bankpaycode" }).first?.value),
            let bankpayValue = (queryItems.filter({ $0.name == "bankpayvalue" }).first?.value) else {
        return
    }

    // Create url to open in webView
    let resPair = (bankpayCode, bankpayValue)
    func makeNiceTransPaymentsQuery(res: (String, String)) -> String {
        if let niceUrl = niceTransUrl, let tid = bankTid {
            let result = "\(niceUrl)" +
                    "?callbackparam2=\(tid)" +
                    "&bankpay_code=\(res.0)" +
                    "&bankpay_value=\(res.1)"
            dlog("makeNiceTransPaymentsQuery \(result)")
            return result
        }
        return ""
    }

    // Open url in webView according to the result code
    if let code = BankPayResultCode.from(resPair.0) {
        switch code {
        case "000":
            print("Success")
            if let url = URL(string: makeNiceTransPaymentsQuery(res: resPair)) {
                var request = URLRequest(url: url)
                request.httpMethod = "POST"
                wkwebview.load(request)
            }
        case "091", "060", "050", "040", "030":
            print("Cancel")
        }
    }
}

// Extension to extract query parameters from the url
extension URL {
    func queryParams() -> [String: Any] {
        let queryItems = URLComponents(url: self, resolvingAgainstBaseURL: false)?.queryItems
        let queryTuples: [(String, Any)] = queryItems?.compactMap {
            guard let value = $0.value else {
                return nil
            }
            return ($0.name, value)
        } ?? []
        return Dictionary(uniqueKeysWithValues: queryTuples)
    }
}
```

