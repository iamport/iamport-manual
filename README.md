# 아임포트  

:globe_with_meridians: [EN](/en/README.md)  

아임포트(I'mport)는,  
PG사 결제모듈에 대한 연동 개발을 진행할 때, 다양한 개발환경에서 보다 쉽고 빠르게 개발할 수 있도록 제공되는 **결제 플랫폼** 혹은 **결제 호스팅 서비스** 입니다.  

결제모듈연동을 위해 불필요한 수고를 해야하는 개발자의 입장에서 만들어진 서비스이기 때문에, 아쉽지만 구매자는 아임포트를 기반으로 만들어진 사이트인지 그렇지 않은 사이트인지 구분하지 못합니다.  
결제 프로세스 도중 카드사가 필요로하는 보안키보드모듈, 방화벽모듈이 있다면 늘 그래왔던 것처럼 내 컴퓨터에 설치해야 하는 것은 똑같다는 것을 의미합니다.  

이것은 PG사와 카드사가 요구하는 인증결제 프로세스를 그대로 준수하도록 아임포트가 설계되었다는 것을 의미하기도 하므로, 아임포트가 PG사나 카드사의 backdoor를 찾아 서비스하는 것은 아닌지 걱정하실 필요가 없습니다.  
카페24, 메이크샵, 고도몰과 같은 쇼핑몰 호스팅 서비스들처럼 아임포트 서버가 가맹점의 서버를 대신해 결제를 위한 통신과정을 모두 마친 후 그 결과를 기술적인 방법으로 알려줄 뿐입니다.  

## 특징  
### 한글처리방식  
**UTF-8**인코딩을 사용합니다. euc-kr 인코딩 변환에 대한 수고가 필요없습니다.  

### javascript API
html form submission 보다는 javascript function 을 사용합니다.  

PG결제창 호출을 위해 hidden input에 데이터를 전달하여 form submit하는 대신,

```html
<form name="pgForm">
	<input type="hidden" name="Amt" value="1000">
	<input type="hidden" name="BuyerName" value="홍길동">
	<input type="hidden" name="OrderName" value="결제테스트">
</form>
```

javascript function을 호출하면 됩니다.  

```javascript
IMP.request_pay({
	amount : 1000,
	buyer_name : '홍길동',
	name : '결제테스트'
}, function(response) {
	//결제 후 호출되는 callback함수
	if ( response.success ) { //결제 성공
		console.log(response);
	} else {
		alert('결제실패 : ' + response.error_msg);
	}
})
```

### REST API  
결제완료 후 결제 결과를 조회할 수 있도록 인증 과정을 거쳐 REST API가 제공됩니다. 때문에, 사용하시는 Backend 프로그래밍 언어와 무관하게 복잡한 설치 프로그램 필요없이 연동이 가능합니다.  


## 결제 방식  
### 인증 결제  
카드정보 입력 및 안심클릭인증 또는 ISP인증 후 결제가 진행되는 일반적인 결제 방식입니다.  

인증프로세스 및 결제프로세스에 대한 보다 상세한 안내는 [인증결제/background.md](인증결제/background.md)를 참고해주세요.  
결제 연동에 대한 상세한 매뉴얼은 [인증결제](인증결제/README.md)를 참고해주세요.  

### 비인증 결제  
안심클릭인증, ISP인증과 같은 일반적인 인증 프로세스 대신, 카드번호+유효기간+생년월일+비밀번호 확인을 통한 간략화된 인증 후 결제가 이루어지는 방식입니다.  
최초 1회 카드정보 등록 후 매월 자동으로 결제가 이뤄져야 하는 **정기구독형태의 과금** 등 특수한 경우에 한하여 제공됩니다.  
보다 상세한 연동 안내는 [빌링키 발급 및 재결제](비인증결제/README.md)를 참고해주세요.  

## 문의 및 안내  
support@iamport.kr 를 통해 보다 상세한 기술문의를 하실 수 있습니다.  
PG계약 및 일반 문의사항은 cs@iamport.kr 로 가능합니다.  