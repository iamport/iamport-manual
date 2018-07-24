# javascript 버전별 안내

### iamport.payment-1.1.5.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.1.5.js" type="text/javascript"></script>
```  
**2017-04-03 배포**  
- 모바일 결제에서 `IMP.request_pay(param)`와 같이 callback function 이 누락된 채 호출되었으나, 결제프로세스 시작 전 사전 필터링 단계에서 실패사유가 발생한 경우에도 `m_redirect_url`로 이동하게 됩니다.  
*(결제프로세스가 시작된 후 성공/실패에 대한 경우에는 이전 버전에서도 `m_redirect_url`로 이동이 이루어지고 있습니다.)*  

##### 결제프로세스 시작 전에 발생할 수 있는 실패 사유   
- 이미 결제된 merchant_uid를 재시도하는 경우
- 결제요청 파라메터가 올바르지 않은 경우

##### 결제프로세스 시작 후 발생할 수 있는 실패 사유  
- 카드 사용 정지, 한도초과  
- 비밀번호 오류 횟수 초과  

### iamport.payment-1.1.4.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.1.4.js" type="text/javascript"></script>
```  
**2016-11-14 배포**  
- Agency-Tier기능 제공, IMP.agency(가맹점식별코드, Tier코드)함수 추가  
- SMS휴대폰본인인증 기능 제공, IMP.certification()함수 추가  

### iamport.payment-1.1.3.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.1.3.js" type="text/javascript"></script>
```  
**2016-07-13 배포**  
- 1.1.2버전 소스리팩토링, 성능개선된 버전  


### iamport.payment-1.1.2.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.1.2.js" type="text/javascript"></script>
```  
**2016-03-09 배포**  
- 복수의 PG설정정보를 호출하는 방법을 보다 개선한 버전  


### iamport.payment-1.1.1.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.1.1.js" type="text/javascript"></script>
```  
**2016-02-19 배포**  
- 동일한 PG의 MID를 여러개 사용할 수 있도록 pg파라메터 설정 규칙 확장. pg : '{PG사}.{MID}'  

### iamport.payment-1.1.0.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.1.0.js" type="text/javascript"></script>
```  
**2016-01-19 배포**  
- 하나의 계정으로 복수의 PG설정을 사용할 수 있도록 pg 파라메터를 추가. pg : '{PG사}'  


### iamport.payment-1.0.0.js

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment-1.0.0.js" type="text/javascript"></script>
```  
**2014-10-24 배포**  
- 최초 배포된 후 안정화된 버전  

### iamport.payment.js  

```html  
<script src="https://cdn.iamport.kr/js/iamport.payment.js" type="text/javascript"></script>
```  
versioning되기 전 사용된 버전. iamport.payment-1.0.0.js와 동일하며 이후 내용 변경 없음  