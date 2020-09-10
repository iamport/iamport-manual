# 아임포트 REST API를 활용한 KCP 키인결제 또는 빌링키 발급  

- General Guide : [API 방식의 정기결제 연동 가이드](https://docs.iamport.kr/implementation/subscription?lang=ko#issue-billing-a)

일반적으로, KCP에서 제공되는 결제창을 통해 키인결제 및 카드등록이 진행됩니다. 다만, 사전 협의를 통해 아임포트 REST API만을 활용한 키인결제 및 카드등록 또한 가능하므로 관련 문의사항은 [아임포트 고객센터(1670-5176)](cs@iamport.kr) 로 연락바랍니다.  
(`/subscribe/payments/onetime`, `/subscribe/customers/{customer_uid}` API 사용 가능)

KCP결제창을 통한 카드등록 안내는 [KCP결제창을 통한 카드등록 예제](/비인증결제/example/kcp-request-billing-key.md) 참고해주세요.  


## 1. PG설정  
아임포트 관리자 페이지의 시스템 설정 > PG설정(인증방식결제)에서 설정합니다.  
![REST API방식의 KCP 빌링 설정](../screenshot/kcp-api-setting.png)

- KCP 선택  
- 사이트코드에 KCP에서 발급받으신 사이트코드 입력  
- 사이트키에 KCP에서 발급받으신 사이트키 입력  
- 배치결제그룹아이디는 [설정 매뉴얼](http://www.iamport.kr/download/kcp-billing.pdf)에 따라 생성 후 입력  

## 2. 키인결제 API  
키인결제는 `POST /subscribe/payments/onetime` API 를 사용해 진행됩니다.  

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"card_number":"1234-1234-1234-1234", "expiry":"2025-12", "birth":"820213", "pwd_2digit":"00", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/onetime
```

## 3. 빌링키 등록요청  
빌링키 발급이 성공적으로 이루어지면, 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 보관됩니다.  

`POST /subscribe/customers/{customer_uid}` API를 통해 빌링키 등록이 가능하며, 최초 과금 결제와 동시에 빌링키 등록이 필요하시다면 `POST /subscribe/payments/onetime` API를 호출하되 `customer_uid` 파라메터를 함께 전송하시면 됩니다.  

### 3.1 빌링키만 등록하는 경우  
```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"card_number":"1234-1234-1234-1234", "expiry":"2025-12", "birth":"820213", "pwd_2digit":"00"}' \
     https://api.iamport.kr/subscribe/customers/your-customer-unique-id
```

### 3.2 최초 과금 결제와 빌링키 등록을 동시에 하는 경우  
```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "card_number":"1234-1234-1234-1234", "expiry":"2025-12", "birth":"820213", "pwd_2digit":"00"}' \
     https://api.iamport.kr/subscribe/payments/onetime
```

## 4. 발급된 빌링키로 결제요청  
빌링키 발급이 성공적으로 이루어지면, 전달된 `customer_uid` 와 1:1 매칭되어 아임포트에 보관됩니다.  
때문에, `customer_uid`를 전달하면 발급된 빌링키를 찾아 결제승인 요청을 진행하게 됩니다.  

```
curl -H "Content-Type: application/json" \   
     -X POST -d '{"customer_uid":"your-customer-unique-id", "merchant_uid":"order_id_8237352", "amount":3000}' \
     https://api.iamport.kr/subscribe/payments/again
```