# 1. PC, 모바일 브라우저 연동  


Paypal의 경우 Paypal 보안정책으로 인해 PC, 모바일 모두 `IMP.request_pay(param, callback)`의 `callback`함수를 사용할 수 없습니다.  
Express Checkout을 위해 Paypal로 페이지 이동(redirection)이 이루어지기 때문에 PC, 모바일 모두 `m_redirect_url`파라메터를 반드시 설정해주셔야 합니다.  

*참고 : 한국 사업자의 경우 Paypal Standard 와 Express Checkout 2가지 결제 방법을 사용할 수 있습니다. 아임포트에서는 사용자 이탈률이 적고 결제데이터 누락 오류가 없는 Express Checkout을 적용하고 있습니다.*


```javascript
IMP.request_pay({
    pg : 'paypal',
    pay_method : 'card',
    merchant_uid : 'merchant_' + new Date().getTime(),
    name : '주문명:결제테스트',
    amount : 14.20,
    currency : 'USD', //paypal에서 KRW지원하지 않음. USD외 다른 currency가 필요하신 경우 고객센터로 연락주세요
    buyer_email : 'iamport@siot.do',
    buyer_name : '구매자이름',
    buyer_tel : '010-1234-5678',
    buyer_addr : '서울특별시 강남구 삼성동',
    buyer_postcode : '123-456',
    m_redirect_url : 'https://www.yourservice.com/payment/complete' //결제완료 후 이동될 페이지 주소를 지정해주세요. query string이 추가되어 전달됩니다.
});
```

# 2. m\_redirect\_url 사용  

보다 상세한 내용은 [인증결제/getstarted.md](../getstarted.md) 의 **2.2.b m\_redirect\_url** 섹션을 참조해주세요.  

`IMP.request_pay(param)` 호출 시 `m\_redirect\_url`을 `https://www.yourservice.com/payment/complete`로 지정하셨다면 실제 이동되는 주소는 다음과 같습니다. 

###결제 성공  
`https://www.yourservice.com/payment/complete?imp_uid={아임포트거래고유번호}&merchant_uid={상점거래고유번호}`  

###결제 실패 
`https://www.yourservice.com/payment/complete?imp_uid={아임포트거래고유번호}&merchant_uid={상점거래고유번호}&error_msg={실패사유}`  

`/payment/complete` 주소를 처리하는 웹서버단 로직에서 `imp_uid` 또는 `merchant_uid`로  아임포트 REST API요청을 통해 `imp_uid` 또는 `merchant_uid`에 대한 결제 상세 정보를 조회하실 수 있습니다.  
