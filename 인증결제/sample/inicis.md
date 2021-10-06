# KG이니시스 일반결제 연동 가이드

:globe_with_meridians: [EN](/en/General/sample/inicis.md)  

본 문서는 KG이니시스 관련 내용만 기술하므로 README 파일에서 다음 일반결제 연동 정보를 확인하세요.

- [PC/모바일 웹에서 PG 연동하기](../README.md#pc-mobile)
- [모바일 앱 WebView에서 PG 연동하기](../README.md#webview)

## 1. PG 설정하기

<a href="https://guide.iamport.kr/8f617f32-564d-464e-9850-bf7648c609b0" target="_blank">KG이니시스 일반결제 테스트 모드 설정</a> 페이지의 내용를 참고하여 PG 설정을 합니다.

## 2. 결제 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/sdk/javascript-sdk#request_pay)을 호출하여 KG이니시스 결제창을 호출합니다.

PC의 경우 `IMP.request_pay(param, callback)` 호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `pg` : 
    - 등록된 PG사가 하나일 경우에는 미 설정시 `기본 PG사`가 자동으로 적용됩니다.
    - 등록된 PG사가 여러개인 경우에는 `html5_inicis` 또는 `inicis`(ActiveX 방식일 경우)로 지정합니다.
	- KG이니시스에서 발급받은 상점아이디가 여러개(각각 일반 및 정기)인 경우에는 `html5_inicis.{상점아이디}` 또는 `inicis.{상점아이디}`(for ActiveX)로 지정합니다.
- `pay_method` : card(신용카드), trans(실시간계좌이체), vbank(가상계좌), 또는 phone(휴대폰소액결제)
- `buyer_tel`: 필수 입력 (미설정 시 이니시스 결제창에서 오류 발생 가능)

```javascript
IMP.request_pay({
    pg : 'html5_inicis',
    pay_method : 'card',
    merchant_uid: "order_no_0001", // 상점에서 관리하는 주문 번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{모바일에서 결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
}, function(rsp) { // callback 로직
	//* ...중략 (README 파일에서 상세 샘플코드를 확인하세요)... *//
});
```

## 3. 모바일 앱 WebView에서 연동하기

### 3.1 iOS

실시간계좌이체 결제 승인 완료 후 다음과 같이 webView에서 최종 결제 요청을 해야 합니다.

```swift
// url 변경시점
open func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction, decisionHandler: @escaping (WKNavigationActionPolicy) -> Void) {
    let myAppScheme = "가맹점 앱의 appscheme"
    if (url.absoluteString.hasPrefix(myAppScheme)) {
        processInicisTrans(appScheme, url)
        return
    }
    .. 생략
}

// 전달받은 url 을 webView 에 POST 로 전달
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

    let m_redirect_url = "m_redirect_url 으로 설정한 값"

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

