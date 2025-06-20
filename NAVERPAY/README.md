# 네이버페이

## 1. 결제형 네이버페이  

가장 최신의 네이버페이 결제 서비스로, 일반 PG처럼 결제 페이지에서 결제 수단 중 하나로 네이버페이를 선택하여 진행되며, 주문형 네이버페이와 달리 실시간으로 결제정보 확인이 가능합니다. 일반결제와 반복결제(정기결제)를 모두 지원합니다.  

[네이버페이 개발자센터](https://developer.pay.naver.com/docs/v2/api)의 연동 가이드에 따라 아임포트 환경에 맞게 구성되었습니다. 연동 개발이 완료되면, 네이버페이 검수팀의 검수를 받아야 합니다. 아임포트 기술지원부서(support@portone.io)로 연락하여 검수요청서 초안을 받아서 요청할 수 있습니다.  

결제형 네이버페이 연동 가이드:

- [연동 가이드](https://developers.portone.io/opi/ko/integration/pg/v1/naver?v=v1)

## 2. 주문형 네이버페이  

주문 단계부터 상품정보와 연동하여 네이버페이로 결제를 진행하는 방식입니다.  

주문형 네이버페이 연동 가이드:  

- [주문형 네이버페이 연동하기](sample/naverpay-order.md)
- [배송비 정책 및 예시](sample/naverpay-shipping.md)
- [naverProducts JSON Schema](sample/naverpay-schema.md)
