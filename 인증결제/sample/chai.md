들어가기에 앞서
===============

차이(Terra) 결제서비스는 차이(Terra)앱을 통한 계좌등록이 필요합니다. 테스트모드 결제는 차이(Terra)에서 배포하는 테스트용 앱을 설치해야하며, FAKE 계좌를 등록하여 실제 출금없이 결제해보실 수 있습니다.

테스트용 차이(Terra)앱
----------------------

-	iOS : [다운로드-TestFlight](https://testflight.apple.com/join/BbNKLjl5)
-	Android : [다운로드](https://static.chai.finance/app_build/app-dev-1.0.999+1100033.apk)

테스트용 FAKE계좌
-----------------

-	k뱅크 : 100107607902
-	국민은행 : 04970204060427
-	우리은행 : 1002370055785
-	우체국 : 01240102227177
-	농협 : 1111111111111
-	SC제일은행 : 86020057408
-	대구은행 : 505102687244
-	부산은행 : 1010000014002

PC / 모바일 웹 연동
===================

리디렉션
--------

`IMP.request_pay(param, callback)`호출을 통해 결제가 시작되면, 차이(Terra)의 서비스페이지로 리디렉션되므로 리디렉션 된 이후에는 callback 함수를 활용하여 결제완료 통지를 받을 수 없습니다.  
때문에, PC / 모바일 모두 `m_redirect_url` 파라메터를 통해 결제가 완료되었을 때 복귀할 페이지 URL을 지정해야 합니다.

`param`에서 지원되는 속성에 대한 상세한 내용은 [아임포트매뉴얼](https://docs.iamport.kr/tech/imp) 에서 확인바랍니다.

호출 파라메터
-------------

차이(Terra) 결제를 지정하기 위한 `pg` 파라메터는 `chai` 입니다. [아임포트 관리자페이지-시스템 설정](https://admin.iamport.kr/settings) 내에 차이(Terra) 설정이 하나인 경우에는 `pg : 'chai'` 파라메터를 통해 결제를 호출하실 수 있습니다.

차이(Terra)의 경우 계약 시 public key를 통해 계정을 구분하므로, [아임포트 관리자페이지-시스템 설정](https://admin.iamport.kr/settings) 내에 차이(Terra) 설정이 여러 개인 경우에는 `pg : 'chai.{public_key}'` 파라메터를 통해 구분하여 호출하실 수 있습니다.

```javascript
IMP.init('imp37739582'); //아임포트 회원가입 후 발급된 가맹점 고유 코드를 사용해주세요. 예시는 차이(Terra) 공식 아임포트 데모 계정입니다.

IMP.request_pay({
    pg : 'chai',
    pay_method : 'trans', //차이(Terra)는 계좌기반의 서비스이므로 trans만 적용가능합니다.
    merchant_uid : 'merchant_' + new Date().getTime(), //상점에서 관리하시는 고유 주문번호를 전달
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : 'https://www.my-service.com/payments/complete/mobile'
}, function(rsp) {
    if ( !rsp.success ) {
    	//차이(Terra) 결제 시작 페이지로 리디렉션되기 전에 오류가 난 경우
        var msg = '오류로 인하여 결제가 시작되지 못하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

앱 내 WebView 연동
==================

WebView를 통하여 웹 방식으로 결제가 진행되므로, 모바일 웹 결제 연동과 동일합니다. 결제를 위하여 차이(Terra) 앱으로 이동하는 경우 원래의 앱으로 복귀하기 위해 `app_scheme` 파라메터가 필요하다는 점만 다릅니다.(타 PG사 연동 시 `app_scheme` 사용법과 동일합니다.)

`app_scheme` 파라메터 형식
--------------------------

아래 2가지 형태를 모두 지원하고 있습니다.

-	iamporttest : URL scheme값만 지정하는 형태로 `://` 을 아임포트에서 지정하므로 `iamporttest://` 이 실제 호출됩니다.
-	iamporttest://path?query : 앱을 통해 전달받을 파라메터를 함께 지정하는 형태로 지정된 `app_scheme` 그대로 호출이 이뤄집니다.

차이(Terra) 앱 이동을 위한 whitelist 등록
-----------------------------------------

사용자의 선택에 따라 휴대폰에 설치된 차이(Terra) 앱으로 이동하여 결제프로세스를 진행할 수 있습니다. iOS의 경우 Info.plist 파일의 `LSApplicationQueriesSchemes` 에 아래의 URL scheme 을 추가해주어야 차이(Terra) 앱으로 이동이 이뤄질 수 있습니다.

-	chaipayment

```xml
<key>LSApplicationQueriesSchemes</key>
<array>
  <!-- 그 외 기타 필요한 scheme 추가 -->
	<string>chaipayment</string>
</array>
```
