# 차이(CHAI) 비인증 결제 연동 가이드

비인증 결제 연동에 대한 자세한 내용은 [정기결제 연동 가이드](https://docs.iamport.kr/implementation/subscription?lang=ko#request-payment)를 참고하세요.

## 1. 준비하기

### 1.1 테스트용 차이 앱 설치하기

차이에서 제공하는 테스트용 앱을 설치합니다.

- Android : [https://appdistribution.firebase.dev/i/8fef29de1f667252](https://appdistribution.firebase.dev/i/8fef29de1f667252)
  - staging 앱을 설치
- iOS : [https://testflight.apple.com/join/ZgQIFce5](https://testflight.apple.com/join/ZgQIFce5)
  - 앱의 오른쪽 엣지에서 왼쪽으로 swipe 후, staging 설정

### 1.2 테스트용 Fake 계좌 등록하기

차이 앱에 테스트용 fake 계좌를 등록합니다.<Br />

여러명의 사용자가 하나의 fake 계좌를 중복으로 등록할 수 있습니다.

![테스트용 fake 계좌 리스트](screenshot/fake-accounts.png)  

### 1.3 PG 설정하기

1. [아임포트 관리자 콘솔 > 시스템 설정 > PG설정(일반결제 및 정기결제)](https://admin.iamport.kr/settings#tab_pg) 탭으로 이동합니다.
1. **PG사**에 `[간편결제] 차이`를 선택합니다.
1. <b>테스트모드(Sandbox)</b>를 `ON`으로 설정합니다.
1. **public_api_key**에 차이에서 발급받은 `public API 키`를 입력합니다.
1. **private_api_key**에 차이에서 발급받은 `private API 키`를 입력합니다.

![아임포트 관리자 콘솔에서 PG설정](screenshot/chai-setting.png)  

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/tech/imp)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다. 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요.

- `pg` : 차이에서 발급받은 상점아이디가 하나인 경우에는 `pg: 'chai'`를, 여러개인 경우에는 `pg: 'chai.{public_key}'`를 입력합니다.
- `amount` : 빌링키 발급과 결제를 같이 하려면 실제 결제할 금액을 입력하고 [발급받은 빌링키로 결제 요청](#request-pay)을 합니다.

```jsx
IMP.init('{가맹점 식별코드}'); // 예: imp37739582(차이 공식 데모 계정용 가맹점 식별코드)

IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans', // 차이는 계좌기반의 서비스이므로 trans만 입력 가능
    merchant_uid : '{고유 주문번호}', // 예: issue_billingkey_monthly_0001
    name : '주문명: 결제 테스트',
    amount : 0, // 결제창에 표시될 금액. 실제 승인은 이뤄지지는 않음
    customer_uid : '{카드(빌링키)와 1:1로 대응하는 값}', // 필수 입력 (예: gildong_0001_1234)
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자 이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : '{결제 완료 후 리디렉션 될 URL}' // 예: https://www.my-service.com/payments/complete/mobile
}, function(rsp) {
    if ( !rsp.success ) {
    	// 차이 결제 시작 페이지로 리디렉션되기 전에 오류가 난 경우
        var msg = '오류로 인하여 요청을 처리하지 못했습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

<a name="request-pay" />

## 3. 빌링키로 결제 요청하기

빌링키 발급이 성공하면 빌링키는 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 저장됩니다. 보안상의 이유로 서버는 빌링키에 직접 접근할 수 없기 때문에 `customer_uid`를 이용해서 다음과 같이 결제 요청을 합니다.

```jsx
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"merchant_xxxxxxxx", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```
