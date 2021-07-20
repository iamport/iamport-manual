# KG이니시스 비인증 결제 연동 가이드

KG이니시스의 웹표준 결제창/모바일 결제창을 통해서 빌링키 발급을 요청하여 발급받은 빌링키로 결제를 요청할 수 있습니다. REST API `/subscribe/payments/onetime`과 `/subscribe/customers/{customer_uid}`는 지원하지 않습니다.

## 1. PG 설정하기

1. [아임포트 관리자 콘솔 > 시스템 설정 > PG설정(일반결제 및 정기결제)](https://admin.iamport.kr/settings#tab_pg) 탭으로 이동합니다.
1. **PG사 추가**를 누르면 나타나는 PG설정 탭의 **PG사**에 `KG이니시스(웹표준결제창)`를 선택합니다.
1. **테스트모드(Sandbox)** 옵션을 `OFF`로 설정합니다.
1. <b>PG상점아이디 (MID)</b>에 `INIBillTst`를 입력합니다.
1. **웹표준결제 signKey**에 `SU5JTElURV9UUklQTEVERVNfS0VZU1RS`를 입력합니다.
1. **빌링용 merchantKey**에 `b09LVzhuTGZVaEY1WmJoQnZzdXpRdz09`를 입력합니다.
1. 하단에 **전체 저장** 버튼을 눌러 설정을 저장합니다.

![아임포트 관리자 콘솔에서 PG설정](../screenshot/inicis-setting.png)

## 2. 빌링키 발급 요청하기

[IMP.request_pay(param, callback)](https://docs.iamport.kr/tech/imp)을 호출하여 빌링키 발급을 위한 결제창을 호출합니다. 자세한 내용은 [일반결제창으로 빌링키 요청하기](https://docs.iamport.kr/implementation/subscription#issue-billing-b)를 참고하세요. PC의 경우 `IMP.request_pay(param, callback)`호출 후 callback으로 실행되고, 모바일의 경우 `m_redirect_url`로 리디렉션됩니다.

- `amount` : 빌링키 발급과 결제를 같이 하려면 실제 결제할 금액을 입력하고 [발급받은 빌링키로 결제 요청](#request-pay)을 합니다.

```javascript
IMP.request_pay({
	pg : "html5_inicis.INIBillTst", // 실제 계약 후에는 실제 상점아이디로 변경
	pay_method : 'card', // 'card'만 지원됩니다.
	merchant_uid : 'merchant_' + new Date().getTime(),
	name : '최초인증결제',
	amount : 0, // 결제창에 표시될 금액. 실제 승인이 이뤄지지는 않습니다. (모바일에서는 가격이 표시되지 않음)
	customer_uid : 'your-customer-unique-id', //customer_uid 파라메터가 있어야 빌링키 발급을 시도합니다.
	buyer_email : 'iamport@siot.do',
	buyer_name : '아임포트',
	buyer_tel : '02-1234-1234'
}, function(rsp) {
	if ( rsp.success ) {
		alert('빌링키 발급 성공');
	} else {
		alert('빌링키 발급 실패');
	}
});
```

<a name="request-pay" />

## 3. 빌링키로 결제 요청하기  

빌링키 발급이 성공하면 빌링키는 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 저장됩니다. 보안상의 이유로 서버는 빌링키에 직접 접근할 수 없기 때문에 `customer_uid`를 이용해서 다음과 같이 결제 요청을 합니다.

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"name": "your-service-name", "customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```
