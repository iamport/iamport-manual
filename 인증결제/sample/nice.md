# 나이스페이먼츠 일반결제 연동 가이드

본 문서는 나이스페이먼츠 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기](../README.md#pc-mobile)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/94861fdf-c937-473b-81f8-eba6bb4cff9f" target="_blank">나이스페이먼츠 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 나이스페이먼츠 결제창을 호출합니다.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.  

- `pg` : 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용되며, 여러개인 경우에는 `jtnet`으로 지정합니다.
- `pay_method` : card(신용카드), trans(실시간계좌이체), vbank(가상계좌), 또는 phone(휴대폰소액결제)
- `niceMobileV2` : 나이스 모바일 신규버전 적용 여부(기본 값: false). 하위 호환성을 위해 제공됩니다.

```javascript
IMP.request_pay({
    pg : 'nice',
    pay_method : 'card',
    merchant_uid: "order_monthly_0001", //상점에서 생성한 고유 주문번호
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}', // 예: https://www.my-service.com/payments/complete/mobile
	niceMobileV2 : true // 신규 모바일 버전 적용 시 설정
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

## 3. 모바일 앱 WebView에서 연동하기

### 3.1 iOS

실시간계좌이체(뱅크페이) 결제 승인 완료 후 `AppDelegate` 또는 `SceneDelegate`를 통해 전달받은 값으로 다음과 같이 webView에서 최종 결제 요청을 해야 합니다.

```swift
// AppDelegate.swift (SwiftUI 사용시)
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
				Iamport.shared.processBankPayPayment(url)
        return true
    }


// SceneDelegate.swift (SwiftUI 사용시)
func scene(_ scene: UIScene, openURLContexts URLContexts: Set<UIOpenURLContext>) {
        if let url = URLContexts.first?.url {
            Iamport.shared.processBankPayPayment(url)
        }
    }
```

```swift
var bankTid: String?
var niceTransUrl: String?

// url 변경시점
open func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
	if(uri.scheme == "kftc-bankpay") {
	    let queryParams = url.queryParams()
	    bankTid = (queryParams["user_key"] as? String)
	    niceTransUrl = (queryParams["callbackparam1"] as? String)
	
	    // 생략.. 뱅크페이 앱 열기 
	}
}

// 뱅크페이 앱 결과를 받은 후 처리
public func processBankPayPayment(_ url: URL) {
    
    // bankpaycode값과 bankpayvalue값을 추출해 각각 bankpay_code와 bankpay_value값으로 전달
    guard let queryItems = URLComponents(string: url.absoluteString)?.queryItems,
            let bankpayCode = (queryItems.filter({ $0.name == "bankpaycode" }).first?.value),
            let bankpayValue = (queryItems.filter({ $0.name == "bankpayvalue" }).first?.value) else {
        return
    }

    // webview 에 호출할 url 만들기
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

    // 결과 코드에 따라 webview 에 호출
    if let code = BankPayResultCode.from(resPair.0) {
        switch code {
        case "000":
            print("성공")
            if let url = URL(string: makeNiceTransPaymentsQuery(res: resPair)) {
                var request = URLRequest(url: url)
                request.httpMethod = "POST"
                wkwebview.load(request)
            }
        case "091", "060", "050", "040", "030":
            print("취소")
        }
    }
}

// url 에서 쿼리파람을 추출하는 extension
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

