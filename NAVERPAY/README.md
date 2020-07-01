# 네이버페이  

## 1. 결제형 네이버페이  

가장 최신의 네이버페이 결제 서비스로, 운영하시는 사이트 내 네이버페이 결제창이 나타나 진행되며, 주문형 네이버페이와 달리 실시간으로 결제정보 확인이 가능합니다.  
일반결제와 반복결제(정기결제)를 모두 지원합니다.   

[네이버페이 개발자센터](https://developer.pay.naver.com/docs/v2/api) 의 연동 가이드에 따라 아임포트 환경에 맞게 구성한 것이며, [아임포트-결제형 네이버페이 연동가이드](https://github.com/iamport/iamport-manual/blob/master/NAVERPAY/sample/naverpay-pg.md) 를 참고해 개발하시면 됩니다.  
연동 개발이 완료되면, 네이버페이 검수팀의 검수를 받으셔야 하므로 아임포트 기술지원부서(support@iamport.kr)로 연락주시면 검수요청서 초안 전달드리겠습니다. 

- [일반결제 연동 가이드](https://github.com/iamport/iamport-manual/blob/master/NAVERPAY/sample/naverpay-pg.md)
- [반복결제/정기결제 연동 가이드](https://github.com/iamport/iamport-manual/blob/master/NAVERPAY/sample/naverpay-recurring.md)

## 2. 주문형 네이버페이  

주문형 네이버페이 연동 방식은 기존 일반 PG, 간편결제 서비스의 연동방식과는 차이가 많아 별도의 매뉴얼로 구성됩니다.  

- 기본 샘플 : [네이버페이 샘플](sample/README.md)
- 배송비 정책 매뉴얼 : [배송비 정책](sample/naverpay-shipping.md)
- JSON-Schema : [결제요청 파라메터 JSON Schema](sample/naverpay-schema.md)
