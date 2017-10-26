# 1. 설치
## 1.1 요구사항
아임포트 javascript는 **jQuery의 기능을 활용**하여 구현되어 있습니다.  
사용되는 모든 jQuery의 기능은 **jQuery 1.0버전부터 제공되는 기본 기능**이므로 원하시는 jQuery의 버전을 사용하시면 됩니다.  

## 1.2 아임포트 javascript 설치
```html
<script type="text/javascript" src="https://service.iamport.kr/js/iamport.payment-x.y.z.js"></script>
```
[**iamport.payment.js 버전별 차이점**](javascript-version.md)

# 2. 시작하기
*결제단계별 흐름을 이해하시면 아래 연동 과정이 보다 쉬울 수 있습니다.*  [한국결제의 특징](background.md)  

[아임포트 관리자 페이지](https://admin.iamport.kr)에서 계정을 생성하시면, PG사 별로 제공되는 테스트 계정으로 직접 테스트 결제까지 해보실 수 있습니다.  
PG사 테스트 계정은 실제 계정과 동일하게 결제승인이 이루어지지만 결제 후 일정 시간 간격으로 또는 매일 자정직전에 일괄 취소됩니다.  
![아임포트 관리자 페이지에서 PG설정](screenshot/getstarted/pg-setting.png)  
테스트할 PG사를 선택하고 저장합니다.  

## 2.1 아임포트로 PG결제창 띄우기<a id="header_2_1"></a>
```javascript
IMP.init('가맹점식별코드'); // 아임포트 관리자 페이지의 "시스템 설정" > "내 정보" 에서 확인 가능
```
`IMP.init('가맹점식별코드')` 은 일찍 호출해두시면 좋습니다. (ex. DOM Ready 혹은 페이지 로딩 후)  

```javascript
IMP.request_pay({
    pg : 'html5_inicis',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
}, function(rsp) {
    if ( rsp.success ) {
        var msg = '결제가 완료되었습니다.';
        msg += '고유ID : ' + rsp.imp_uid;
        msg += '상점 거래ID : ' + rsp.merchant_uid;
        msg += '결제 금액 : ' + rsp.paid_amount;
        msg += '카드 승인번호 : ' + rsp.apply_num;
    } else {
        var msg = '결제에 실패하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;
    }

    alert(msg);
});
```
`IMP.request_pay(param, callback)` 는 2개의 argument를 받는 함수입니다.  

### 2.1.1 param 속성(공통 속성)
| 속성명 | 타입(typeof) | 설명 | 기본값  | 비고 | 지원버전 |
|---|---|---|---|---|---|
| pg <sup>(*example*)</sup> | string | 하나의 아임포트계정으로 여러 PG를 사용할 때 구분자 | undefined | (선택항목) 누락되거나 매칭되지 않는 경우 아임포트 관리자페이지에서 설정한 "기본PG"가 호출됨<br>**"kakao"**, **"html5\_inicis"** 와 같이 **{PG사명}** 만 지정, **"html5\_inicis.INIpayTest"** 와 같이 **{PG사명}.{상점아이디}** 로 지정<br><br>**html5_inicis**(이니시스웹표준)<br>**inicis**(이니시스ActiveX결제창)<br>**uplus**(LGU+)<br>**nice**(나이스페이)<br>**jtnet**(JTNet)<br>**kakao**(카카오페이)<br>**danal**(다날휴대폰소액결제)<br>**danal_tpay**(다날일반결제)<br>**mobilians**(모빌리언스 휴대폰소액결제)<br>**syrup**(시럽페이)<br>**payco**(페이코)<br>**paypal**(페이팔)<br>**eximbay**(엑심베이)<br> | 1.1.0 부터 |
| pay_method | string | 결제수단 | card | **card**(*신용카드*)<br>**trans**(*실시간계좌이체*)<br>**vbank**(*가상계좌*)<br>**phone**(*휴대폰소액결제*)<br>**samsung**(*삼성페이 / 이니시스 전용*)<br>**kpay**(*KPay앱 직접호출 / 이니시스 전용*)<br>**cultureland**(*문화상품권 / 이니시스, LGU+ 전용*)<br>**smartculture**(*스마트문상 / 이니시스, LGU+ 전용*)<br>**happymoney**(*해피머니 / 이니시스 전용*)<br>**booknlife**(*도서문화상품권 / LGU+ 전용*) | 1.0.0부터 |
| escrow | boolean | 에스크로 결제여부 | false | (선택항목) 에스크로가 적용되는 결제창을 호출 | 1.0.0부터 |
| merchant_uid | string | 가맹점에서 생성/관리하는 고유 주문번호  | random | (필수항목) 결제가 된 적이 있는 merchant_uid로는 재결제 불가  | 1.0.0부터 |
| name | string | 주문명 | undefined | (선택항목) 원활한 결제정보 확인을 위해 입력 권장 (PG사마다 차이가 있지만) 16자이내로 작성하시길 권장 | 1.0.0부터 |
| amount | number | 결제할 금액 | undefined | (필수항목) | 1.0.0부터 |
| tax_free | number | amount 중 면세공급가액 | undefined | (선택항목) 면세공급가액을 지정합니다. amount 중에서 면세공급가액에 해당하는 금액을 지정합니다. (부가세는 (amount - tax_free) / 11 로 계산됩니다) | 1.0.0부터 |
| ~~vat~~ | ~~number~~ | ~~amount 중 부가세 금액~~ | undefined | (Deprecated) 복합과세 적용시 더 정확한 계산을 위해 tax_free파라메터 사용을 권장합니다.~~부가세금액을 지정합니다. vat와 무관하게 amount는 고객으로부터 결제될 금액을 의미합니다.~~ | 1.0.0부터 |
| currency | string | 화폐단위 | KRW<br>(페이팔의 경우에는 USD가 기본값) | (선택항목) KRW / USD / EUR / JPY<br>(참조)페이팔정책상 KRW는 지원되는 결제화폐가 아니므로 USD가 기본 적용됩니다. | 1.0.0부터 |
| language | string | 결제창의 언어 설정 | ko | (선택항목) 구매자에게 제공되는 결제창 화면의 언어 설정<br>*1. KG이니시스, LGU+, 나이스페이먼츠 : 아래 값으로 적용 가능(KG이니시스, 나이스페이먼츠는 PC결제창만 지원됨)*<br>**en** 또는 **ko**<br>*2. Paypal의 경우 2자리 코드 적용*<br>[Paypal 언어 설정 코드 참조](https://developer.paypal.com/docs/classic/api/locale_codes/) | 1.0.0부터 |
| buyer_name | string | 주문자명 | undefined | (선택항목) | 1.0.0부터 |
| buyer_tel | string | 주문자 연락처 | undefined | (필수항목) 누락되거나 blank일 때 일부 PG사에서 오류 발생 | 1.0.0부터 |
| buyer_email | string | 주문자 Email | undefined | (선택항목) | 1.0.0부터 |
| buyer_addr | string | 주문자 주소 | undefined | (선택항목) | 1.0.0부터 |
| buyer_postcode | string | 주문자 우편번호 | undefined | (선택항목) | 1.0.0부터 |
| custom_data | object | 가맹점 임의 지정 데이터  | undefined | (선택항목)주문건에 대해 부가정보를 저장할 공간이 필요할 때 사용. json notation(string)으로 저장됨 | 1.0.0부터 |
| notice_url | string / array of string | Notification URL | undefined | (선택항목) 아임포트 관리자 페이지에서 설정하는 Notification URL을 overwrite할 수 있음. 주문마다 다른 Notification URL이 필요하거나 복수의 Notification URL이 필요한 경우 사용 | 1.0.0부터 |
| display | object | 결제화면과 관련한 옵션 설정 | undefined | (선택항목) 구매자에게 제공되는 결제창 화면에 대한 UI옵션. 2.1.1.a참조 | 1.0.0부터 |


#### 2.1.1.a display 속성  
| 속성명 | 타입(typeof) | 설명 | 기본값  | 비고 | 지원버전 |
|---|---|---|---|---|---|
| card_quota | array of integer | 할부개월수 선택 UI제어옵션 | undefined | (선택항목) 50,000원 이상금액 결제 시 PG사 결제창에서 선택할 수 있는 할부개월목록 UI 제어. <br>1) undefined이면 PG사가 기본 제공하는 할부개월 수 목록을 출력 <br>2) []이면 일시불만 결제 가능 <br>3) [2,3,4,5,6]이면 일시불 + 2,3,4,5,6개월까지 할부개월 선택 가능<br><br>*현재, KG이니시스, KCP만 지원하는 옵션*  | 1.0.0부터 |

### 2.1.2 param 속성(특정 상황에만 필요한 속성)
| 속성명 | 타입(typeof) | 설명 | 기본값  | 비고 | 지원버전 |
|---|---|---|---|---|---|
| digital | boolean | 결제상품이 컨텐츠인지 여부 | false | (선택항목) 휴대폰소액결제시 필수. 반드시 실물/컨텐츠를 정확히 구분해주어야 함 | 1.0.0부터 |
| vbank_due | string | 가상계좌 입금기한 | undefined | (선택항목) YYYYMMDDhhmm 형식 | 1.0.0부터 |
| m\_redirect\_url<sup>(*example*)</sup> | string | 모바일 결제 후 이동될 주소 | undefined | (선택항목) 모바일 결제시에만 사용됨 | 1.0.0부터 |
| app\_scheme<sup>(*example*)</sup> | string | 모바일 앱 결제도중 앱복귀를 위한 URL scheme | undefined | (선택항목) WebView 결제시 필수. ISP/앱카드 앱에서 결제정보인증 후 원래 앱으로 복귀할 때 사용됨 | 1.0.0부터 |
| biz\_num | string | 계약된 사업자등록번호 10자리(기호를 포함하면 안됨) | undefined | (선택항목) 다날-가상계좌 결제시 반드시 설정되어야 합니다. | 1.0.0부터 |

### 2.1.3 m\_redirect\_url  
`m_redirect_url`은 모바일 결제프로세스가 시작되면서 PG사의 페이지로 redirect되었다가, 완료 후 다시 사이트로 복귀하기 위해 사용되는 파라메터입니다.
이 경우, `m_redirect_url`에 해당되는 서버 핸들러에서 결제여부 체크 및 금액 변조확인이 이루어져야 합니다. 이를 위해 결제완료 후 랜딩되는 URL은 다음과 같은 추가 파라메터를 가지게 됩니다.  

- imp_uid
- merchant_uid
- imp_success(@Deprecated. 참조 용도로만 사용 하시고 결제검증에 사용하시면 안됩니다)

```
{
	m_redirect_url : 'http://www.iamport.kr/mobile/landing'
}
```
으로 지정하셨다면, 실제 랜딩되는 주소는 다음과 같습니다.  

```
http://www.iamport.kr/mobile/landing?imp_uid={imp_uid}&merchant_uid={merchant_uid}&imp_success={true/false}
```

query string이 포함된 `m_redirect_url`도 사용하실 수 있습니다.  

```
{
	m_redirect_url : 'http://www.iamport.kr/mobile/landing?utm_source=mobile'
}
```

으로 지정하셨다면, 실제 랜딩되는 주소는 다음과 같습니다.  

```
http://www.iamport.kr/mobile/landing?utm_source=mobile&imp_uid={imp_uid}&merchant_uid={merchant_uid}&imp_uid={true/false}
```

#### Since iamport.payment-1.1.5.js  
iamport.payment-1.1.5.js부터는, 모바일 결제에서 `IMP.request_pay(param)`와 같이 callback function 이 누락된 채 호출되었으나, 결제프로세스 시작 전 사전 필터링 단계에서 실패사유가 발생한 경우에도 `m_redirect_url`로 이동하게 됩니다.  
(결제프로세스가 시작된 후 성공/실패에 대한 경우에는 이전 버전에서도 `m_redirect_url`로 이동이 이루어지고 있습니다.)  

##### 결제프로세스 시작 전에 발생할 수 있는 실패 사유   
- 이미 결제된 merchant_uid를 재시도하는 경우
- 결제요청 파라메터가 올바르지 않은 경우

##### 결제프로세스 시작 후 발생할 수 있는 실패 사유  
- 카드 사용 정지, 한도초과  
- 비밀번호 오류 횟수 초과  


### 2.1.4 callback의 구성  
```javascript
function(rsp) {
	if ( rsp.success ) {
        var msg = '결제가 완료되었습니다.';
        msg += '고유ID : ' + rsp.imp_uid;
        msg += '상점 거래ID : ' + rsp.merchant_uid;
        msg += '결제 금액 : ' + rsp.paid_amount;
        msg += '카드 승인번호 : ' + rsp.apply_num;
    } else {
        var msg = '결제에 실패하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;
    }

    alert(msg);
}
```
| rsp 속성명 | 타입(typeof) | 설명 | 비고 |
|---|---|---|---|
|success|boolean|결제처리가 성공적이었는지 여부|실제 결제승인이 이뤄졌거나, 가상계좌 발급이 성공된 경우, true|
|error_code|string|결제처리에 실패한 경우 단축메세지|현재 코드체계는 없음|
|error_msg|string|결제처리에 실패한 경우 상세메세지||
|imp_uid|string|아임포트 거래 고유 번호|아임포트에서 부여하는 거래건 당 고유한 번호(success:false일 때, 사전 validation에 실패한 경우라면 imp_uid는 null일 수 있음)|
|merchant_uid|string|가맹점에서 생성/관리하는 고유 주문번호||
|pay_method|string|결제수단|**card**(*신용카드*), **trans**(*실시간계좌이체*), **vbank**(*가상계좌*), **phone**(*휴대폰소액결제*)|
|paid_amount|number|결제금액|실제 결제승인된 금액이나 가상계좌 입금예정 금액|
|status|string|결제상태|**ready**(*미결제*), **paid**(*결제완료*), **cancelled**(*결제취소, 부분취소포함*), **failed**(*결제실패*)|
|name|string|주문명||
|pg_provider|string|결제승인/시도된 PG사|**html5_inicis**(*웹표준방식의 KG이니시스*), **inicis**(*일반 KG이니시스*), **kakao**(*카카오페이*), **uplus**(*LGU+*), **nice**(*나이스정보통신*), **jtnet**(*JTNet*), **danal**(*다날*)|
|pg_tid|string|PG사 거래고유번호||
|buyer_name|string|주문자 이름||
|buyer_email|string|주문자 Email||
|buyer_tel|string|주문자 연락처||
|buyer_addr|string|주문자 주소||
|buyer_postcode|string|주문자 우편번호||
|custom_data|object|가맹점 임의 지정 데이터||
|paid_at|number|결제승인시각|UNIX timestamp|
|receipt_url|string|PG사에서 발행되는 거래 매출전표 URL|전달되는 URL을 그대로 open하면 매출전표 확인가능|

| rsp 속성명 | 타입(typeof) | 설명 | 비고 |
|---|---|---|---|
|apply_num|string|카드사 승인번호|신용카드결제에 한하여 제공|
|vbank_num|string|가상계좌 입금계좌번호|PG사로부터 전달된 정보 그대로 제공하므로 숫자 외 dash(-)또는 기타 기호가 포함되어 있을 수 있음|
|vbank_name|string|가상계좌 은행명||
|vbank_holder|string|가상계좌 예금주| **계약된 사업자명으로 항상 일정함**. 단, 일부 PG사의 경우 null반환하므로 자체 처리 필요 |
|vbank_date|number|가상계좌 입금기한|UNIX timestamp|


## 2.2 결제완료 시 서버에 알려주기
[2.1](#header_2_1)에서 호출된 결제창을 통해 구매자가 결제 프로세스를 완료하면, **아임포트 서버는 가맹점의 서버를 대신하여 PG사에 필요한 승인요청을 한 후 결과를 아임포트 서버에 저장**을 합니다.  
대부분의 경우 가맹점에서도 해당 결제건에 대해 **"결제상태"** 혹은 **"결제완료여부"** 를 비롯하여 결제정보를 데이터베이스에 기록할 필요가 있으므로, **가맹점 서버가 아임포트 서버로부터 관련 정보를 조회/확인할 수 있도록 결제 후 프로세스**가 구현되어야 합니다.
### 2.2.a PC버전<a id="header_2_2_a"></a>
PC결제의 경우, 결제프로세스가 종료된 후 `IMP.request_pay(param, callback)`의 callback함수가 실행이 됩니다. callback함수 내에서, 가맹점 서버로 결제정보 확인에 필요한 파라메터(imp_uid, merchant_uid)를 전달하여 가맹점 서버에서 REST API를 실행합니다.

1. `IMP.request_pay(param, callback)` 호출로 결제창 호출
2. 결제 후 callback이 실행되었을 때 가맹점 서버로 **imp\_uid** 전달
3. 가맹점 서버에서 REST API로 결제정보 조회 (예시.[`https://api.iamport.kr/payments/{imp_uid}`](https://api.iamport.kr/#!/payments/getPaymentByImpUid))
4. 조회된 결제 정보가 `status == 'paid' && amount == {결제되었어야 할 금액}` 조건에 해당되는지 체크
5. 가맹점 DB에 필요한 정보 기록 및 서비스 루틴 수행

#### javascript(in browser)

```javascript
IMP.request_pay({
    pg : 'inicis',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456'
}, function(rsp) {
    if ( rsp.success ) {
    	//[1] 서버단에서 결제정보 조회를 위해 jQuery ajax로 imp_uid 전달하기
    	jQuery.ajax({
    		url: "/payments/complete", //cross-domain error가 발생하지 않도록 동일한 도메인으로 전송
    		type: 'POST',
    		dataType: 'json',
    		data: {
	    		imp_uid : rsp.imp_uid
	    		//기타 필요한 데이터가 있으면 추가 전달
    		}
    	}).done(function(data) {
    		//[2] 서버에서 REST API로 결제정보확인 및 서비스루틴이 정상적인 경우
    		if ( everythings_fine ) {
    			var msg = '결제가 완료되었습니다.';
    			msg += '\n고유ID : ' + rsp.imp_uid;
    			msg += '\n상점 거래ID : ' + rsp.merchant_uid;
    			msg += '\결제 금액 : ' + rsp.paid_amount;
    			msg += '카드 승인번호 : ' + rsp.apply_num;

    			alert(msg);
    		} else {
    			//[3] 아직 제대로 결제가 되지 않았습니다.
    			//[4] 결제된 금액이 요청한 금액과 달라 결제를 자동취소처리하였습니다.
    		}
    	});
    } else {
        var msg = '결제에 실패하였습니다.';
        msg += '에러내용 : ' + rsp.error_msg;

        alert(msg);
    }
});
```

#### server (/payments/complete에 대한 request를 처리하는 루틴)  

*의사코딩이므로 사용하시는 프로그래밍 언어에 맞게 변형하세요*  

```
imp_uid = extract_POST_value_from_url('imp_uid') //post ajax request로부터 imp_uid확인

payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' AND payment_result.amount == amount_to_be_paid
	success_post_process(payment_result) //결제까지 성공적으로 완료
ELSE IF payment_result.status == 'ready' AND payment.pay_method == 'vbank'
	vbank_number_assigned(payment_result) //가상계좌 발급성공
ELSE
	fail_post_process(payment_result) //결제실패 처리
```


### 2.2.b 모바일 버전(브라우저, WebView공통)<a id="header_2_2_b"></a>
일부를 제외한 국내 대부분의 PG사들은 **모바일 결제가 시작되면 페이지를 이동(redirect)시켜버리는 특징이 있습니다. 이 과정에서 기존 페이지가 unload되고 `IMP.request_pay(param, callback)`의 callback함수가 메모리에서 해제**되어버려 결제 완료시 callback응답을 받을 수 없는 상태가 됩니다.  

이와 같은 결제 프로세스에서는 결제를 위해 이동된 페이지(PG사 페이지, 카드사 페이지 등)가 끝나고 다시 원래 서비스로 원상복귀할 수 있는 기능이 제공됩니다.  
`IMP.request_pay(param, callback)` param 중 `m_redirect_url`라는 파라메터가 그 역할을 하게 됩니다.

1. `IMP.request_pay(param, callback)` 호출로 결제창 호출
2. PG사 페이지로 이동(redirect)하면서 결제 프로세스 시작
3. 결제 후 `m_redirect_url`에 지정된 주소로 페이지 이동. 이 때, `m_redirect_url`에 query string으로 **imp\_uid** 와 **merchant\_uid** 가 추가된 주소로 이동됨. (예시. https://www.myservice.com/payment/mobile/success로 지정하면 https://www.myservice.com/payment/mobile/success?imp_uid=imp1234567890&merchant_uid=order12345 로 이동됨)
4. 가맹점 서버(https://www.myservice.com/payment/mobile/success에 대한 처리를 하는 서버로직)에서 query string에 포함된 imp_uid값 또는 merchant_uid값을 추출하여 REST API로 결제정보 조회 (예시.[`https://api.iamport.kr/payments/{imp_uid}`](https://api.iamport.kr/#!/payments/getPaymentByImpUid))
5. 조회된 결제 정보가 `status == 'paid' && amount == {결제되었어야 할 금액}` 조건에 해당되는지 체크
6. 가맹점 DB에 필요한 정보 기록 및 서비스 루틴 수행

#### javascript(in browser)

```javascript
IMP.request_pay({
    pg : 'inicis',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14000,
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : 'https://www.myservice.com/payments/complete'
});
```

PC와 달리 `IMP.request_pay(param)`호출 시 callback함수를 지정하지 않아도 됩니다.  
실제 redirect되는 주소는 m\_redirect\_url에 2개의 query string이 추가되어있습니다.  

- imp_uid(string)
- merchant_uid(string)

즉, 위 예제의 경우 `https://www.myservice.com/payments/complete?imp_uid=xxxxxxx&merchant_uid=yyyyyyy`로 redirect됩니다.  


#### server (https://www.myservice.com/payments/complete에 대한 request를 처리하는 루틴)  

*의사코딩이므로 사용하시는 프로그래밍 언어에 맞게 변형하세요*  

```
imp_uid = extract_GET_value_from_url('imp_uid') //GET query string으로부터 imp_uid확인
//merchant_uid = extract_GET_value_from_url('merchant_uid') //또는, GET query string으로부터 merchant_uid확인

payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' AND payment_result.amount == amount_to_be_paid
	success_post_process(payment_result) //결제까지 성공적으로 완료
ELSE IF payment_result.status == 'ready' AND payment.pay_method == 'vbank'
	vbank_number_assigned(payment_result) //가상계좌 발급성공
ELSE
	fail_post_process(payment_result) //결제실패 처리
```

### 2.2.c 예외
[2.2.b 모바일 버전](#header_2_2_b)에서 설명한 것과 같이 결제 시작과 동시에 페이지 이동이 이루어짐으로써 `m_redirect_url` 파라메터를 사용해야만하는 PG사들이 많지만, 일부 PG사들은 이와 같은 별도의 처리 필요없이 [2.2.a PC버전](#header_2_2_a)와 마찬가지로 callback을 그대로 활용할 수 있는 PG사들도 있습니다.  

#### 모바일에서도 PC와 같이 callback을 사용할 수 있는 PG사
이 경우 PC버전과 모바일버전을 구분할 필요없이 동일한 코드로 처리가 가능합니다.  

- 카카오페이
- 다날
- 모빌리언스

#### 모바일 결제 시, `m_redirect_url`을 사용해야하는 PG사
- KG이니시스(웹표준결제, 일반결제 동일)
- LGU+
- NHN KCP
- 나이스정보통신
- JTNet

### 2.2.d PG사별 연동 샘플코드 보기  
대부분 아임포트의 표준코드를 모든 PG사에 공통으로 사용할 수 있지만 모바일 결제에 있어 일부 예외사항들이 있어 PG사별로 정확한 샘플을 확인해보시는 것이 필요합니다.  

- [KG이니시스](sample/inicis.md)
- [LGU+](sample/uplus.md)
- [NHN KCP](sample/kcp.md)
- [나이스정보통신](sample/nice.md)
- [JTNet](sample/jtnet.md)
- [카카오페이](sample/kakao.md)
- [다날(휴대폰소액결제)](sample/danal.md)
- [KICC(한국정보통신)](sample/kicc.md)

## 2.3 Notification URL(가상계좌 입금통보 포함)  
구매자의 결제 환경에 대해서는 100%장담하기가 어려운 것이 현실입니다.  

특히, 모바일 결제의 경우 `2.1.3`처럼 결제완료 시점을 포착하는 것은 `m_redirect_url`에 전적으로 의존할 수 밖에 없는데, 만약 구매자의 브라우저가 `m_redirect_url` 로 redirect를 하기 전에 비정상적으로 종료가 된다거나 네트웍불안정의 이유로 redirect요청에 실패한다거나 하게 되면 **아임포트 서버는 "결제완료" 상태로 바뀌어있으나 가맹점의 서버는 "미결제" 상태**로 남아있는 문제가 있을 수 있습니다.  

![Notification 개념도](screenshot/background/notification.png)

이런 경우를 대비한 Backup routine으로 제공되는 기능이 Notification입니다. Notification 은 [Webhook](https://en.wikipedia.org/wiki/Webhook) 방식으로써, **아임포트 서버와 가맹점 서버가 데이터 동기화를 할 필요가 있을 때** 호출됩니다.  

- 매출이 발생했을 때(신용카드가 결제완료되었을 때, 가상계좌에 실제 입금이 되었을 때 등)
- 가상계좌가 발급이 되었을 때
- 아임포트 관리자 페이지에서 결제 환불이 수행되었을 때

아임포트 서버가 웹 클라이언트 역할을 수행하게 되며, 지정된 URL로 다음과 같은 POST요청을 보내게 됩니다.  

```
curl -H "Content-Type: application/json" \
    -X POST -d '{"imp_uid":"imp_1234567890", "merchant_uid":"order_id_8237352", "status":"paid"}' \
    {NotificationURL}
```

Notification을 수신할 URL은 [아임포트 관리자 페이지](https://admin.iamport.kr)에서 시스템 설정 > PG연동 설정 하단에서 지정하실 수 있으며, 결제 건 별로 다른 URL을 사용하고 싶거나 여러 URL로의 수신이 필요한 경우에는 2.1.1 의 notice\_url을 활용하시면 됩니다.  
(관리자 페이지에서 지정된 URL을 우선 적용하나, notice\_url이 선언된 경우에 한해 notice_url로 대체합니다)  

#### Content-Type  
아임포트 관리자 페이지에서 Notification 에 대한 Content-Type을 지정하실 수 있습니다.  

- application/x-www-form-urlencoded
- application/json

## 2.4 REST API로 결제정보 확인하기  
2.2, 2.3에서 설명한 방식을 활용해 가맹점의 서버는 결제가 완료되었다는 사실을 인지할 수 있습니다.  
다만, 결제 프로세스가 시작되면 결제해야할 금액 정보를 *(가맹점 서버를 거치지 않고)* 구매자의 브라우저로부터 PG사에 직접 전달하는 구조이므로, 결제가 실제 완료된 시점에 가맹점 서버는 결제되었어야 할 금액이 변경없이 제대로 결제되었는지를 확인하는 것이 보다 안전합니다.
(Notification URL은 공개된 URL이므로 아임포트 이외에도 요청을 할 소지가 있습니다)    

*보다 자세한 사항은 [1. 인증과 결제](background.md)를 참고해주세요*  


[2.2.a](#header_2_2_a) 또는 [2.2.b](#header_2_2_b)를 통해 imp_uid가 가맹점 서버에 전달이 되면, 이 값을 이용해 가맹점 서버는 아임포트 서버로 결제상태, 결제금액 등 상세정보를 요청하는 REST API를 호출합니다.  

REST API는 가맹점 별로 부여된 아임포트 API Key / Secret정보를 활용해 인증을 거쳐야하므로 아임포트는 API request를 신뢰할 수 있는 요청으로 간주하고, 가맹점은 API로부터 수신된 응답정보를 신뢰하고 후속처리를 진행할 수 있습니다.  
**(주의 : API Key / Secret이 노출되기때문에 클라이언트 사이드에서 REST API를 호출하시면 안됩니다. 어차피 cross-domain오류로 인증도 안될 것입니다. )**  

[`https://api.iamport.kr/payments/{imp_uid}`](https://api.iamport.kr/#!/payments/getPaymentByImpUid)에 대한 응답으로 *merchant\_uid, pay\_method, pg\_provider, amount, cancel\_amount, status, paid\_at*  등 다양한 정보가 전달됩니다.  
이 중에서, status와 amount값을 통해 실제 결제가 최종적으로 완료가 되었는지, 원하는 금액만큼 결제가 되었는지 확인할 필요가 있습니다.

```
payment_result = rest_api_to_find_payment(imp_uid) //imp_uid로 아임포트로부터 결제정보 조회
amount_to_be_paid = query_amount_to_be_paid(payment_result.merchant_uid) //결제되었어야 하는 금액 조회. 가맹점에서는 merchant_uid기준으로 관리

IF payment_result.status == 'paid' AND payment_result.amount == amount_to_be_paid
	success_post_process(payment_result) //결제까지 성공적으로 완료
ELSE IF payment_result.status == 'ready' AND payment.pay_method == 'vbank'
	vbank_number_assigned(payment_result) //가상계좌 발급성공
ELSE
	fail_post_process(payment_result) //결제실패 처리
```

서버단에서 REST API를 요청하기 위한 언어별 클라이언트 모듈은, 아임포트를 실제 사용해주시는 가맹점의 개발자분들께서 오픈소스로 공개해주시고 있습니다. 많은 참여 부탁드립니다.   

[github바로가기](https://github.com/iamport/iamport-rest-client)

# 3. 모바일 브라우저와 모바일 WebView의 차이
결제수단별 인증을 위해 3rd-party 앱(ISP앱, 앱카드, BankPay)과 연동을 할 때,

1. My앱 -> 3rd-party앱으로 이동
2. 3rd-party앱 -> My앱으로 이동

처리를 위한 로직이 필요합니다.  

### 대표적인 3rd-party앱의 URL scheme

- ispmobile
- kftc-bankpay
- vguard
- droidxantivirus
- ansimclick
- cardusim
- lottesmartpay
- lotteappcard
- kb-acp
- ansimclickscard
- mpocket.online.ansimclick
- smshinhanansimclick
- shinhan-sr-ansimclick
- hdcardappcardansimclick
- smhyundaiansimclick
- cloudpay
- nhappcardansimclick
- kakaotalk

## 3.1 My앱 -> 3rd-party앱으로 이동
### 3.1.a 안드로이드  
URL scheme을 사용해 3rd-party앱으로 이동이 일어나므로 My앱의 WebView에서 해당 URL scheme에 맞는 동작을 처리할 수 있어야 합니다.

```java

@Override
public boolean shouldOverrideUrlLoading(WebView view, String url) {
	if (!url.startsWith("http://") && !url.startsWith("https://") && !url.startsWith("javascript:")) {
		//3rd-party앱에 대한 URL scheme 대응
		Intent intent = null;

		try {
			intent = Intent.parseUri(url, Intent.URI_INTENT_SCHEME); //IntentURI처리
			Uri uri = Uri.parse(intent.getDataString());

			activity.startActivity(new Intent(Intent.ACTION_VIEW, uri));
			return true;
		} catch (URISyntaxException ex) {
			return false;
		} catch (ActivityNotFoundException e) {
			if ( intent == null )	return false;

			//설치되지 않은 앱에 대해 market이동 처리
			if ( handleNotFoundPaymentScheme(intent.getScheme()) )	return true;

			//handleNotFoundPaymentScheme()에서 처리되지 않은 것 중, url로부터 package정보를 추출할 수 있는 경우 market이동 처리
			String packageName = intent.getPackage();
			if (packageName != null) {
				activity.startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("market://details?id=" + packageName)));
				return true;
			}

			return false;
		}
	}
}
```

### 3.1.b iOS
현재 소스코드 정리 후 github에 공개하려고 준비 중입니다.  

## 3.2 3rd-party앱 -> My앱
My앱에서 정의한 URL scheme을 결제가 완료된 시점에 PG사로부터 호출받음으로써 3rd-party앱으로부터 My앱으로 포커스를 이동시킵니다.

### 결제창 호출 시 app\_scheme파라메터 추가

```javascript
IMP.request_pay({
	/*...중략... */
	app_scheme: 'iamporttest' //예시 URL scheme으로 iamporttest를 사용
})
```

### 3.2.a 안드로이드

#### AndroidManifest.xml에 My앱에서 사용할 URL scheme 정의

```xml
<activity
	android:name=".MainActivity"
	android:label="@string/app_name">

	<intent-filter>
		<action android:name="android.intent.action.VIEW" />
		<category android:name="android.intent.category.DEFAULT" />
		<category android:name="android.intent.category.BROWSABLE" />

		<data android:scheme="iamporttest" /> <!-- 예시로 iamporttest로 설정. My앱의 특징을 나타내는 고유의 scheme을 사용하세요 -->
	</intent-filter>

</activity>
```

#### URL scheme에 의해 실행된 Activity에서 Intent처리  
인증 후 3rd-party앱이 URL scheme에 해당하는 Intent를 호출함으로써  My앱의 지정된 Activity가 실행됩니다.  
**URL scheme뒤에는 WebView가 이동해야 할 Web URL이 파라메터로 전달**되는데 이것을  `WebView.loadUrl(redirectURL)`하면 WebView는 최종적으로 `m_redirect_url`로 랜딩됩니다. *(브라우저와 마찬가지로 `imp_uid`와 `merchant_uid`가 query string으로 포함)*  

예시) `iamporttest://://https://web.nicepay.co.kr/smart/card/isp/ispResult.jsp?tid=nictest00m01011604160159435894`


```java
private final String APP_SCHEME = "iamporttest://"; //AndroidManifest.xml에서 정의한 것과 동일한 URL scheme사용(substring을 위한 용도)

@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	/*...중략...*/

	Intent intent = getIntent();
	Uri intentData = intent.getData();

	if ( intentData != null ) {
		//isp 인증 후 복귀했을 때 결제 후속조치
		String url = intentData.toString();
		if ( url.startsWith(APP_SCHEME) ) {
			//My앱의 WebView가 표시해야 할 웹 컨텐츠의 주소가 전달됩니다.
			String redirectURL = url.substring(APP_SCHEME.length()+3);
			mainWebView.loadUrl(redirectURL);
		}
	}
}
```

#### 참고(예외사항)
안드로이드는 Activity가 종료되면 Activity Stack(Task)에서 이전 Activity로 자동으로 이동하는 특성이 있습니다.  
이러한 특성을 활용하여 KG이니시스의 경우, URL scheme을 사용하지 않아도 동작에 문제가 없도록 설계되어있습니다.

##### PG사별 샘플보기
- [KG이니시스](https://github.com/iamport/iamport-inicis-android)
- [나이스정보통신](https://github.com/iamport/iamport-nice-android)
- [카카오페이](https://github.com/iamport/iamport-kakao-android)

### 3.2.b iOS  

현재 소스코드 정리 후 github에 공개하려고 준비 중입니다.
