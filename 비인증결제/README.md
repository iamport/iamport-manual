# PG사별 정기결제(빌링) 연동 가이드

:globe_with_meridians: [EN](/en/Subscription/README.md)

구독형 정기결제, 종량제 과금결제 등 원하는 시점에 재결제를 하기 위해서 카드사에 고객의 카드정보를 전달하여 카드정보와 매칭되는 결제용 암호화 키인 빌링키를 발급받아야 합니다.<Br />

빌링키 발급을 위해서 다음과 같이 두 가지 방식으로 카드정보를 카드사에 전달할 수 있습니다.

- **결제창 방식**: PG사의 결제창을 통해서 카드정보를 전달하는 방식. 
- **REST API 방식**: 아임포트 REST API를 통해서 카드정보를 전달하는 방식.

| ℹ️  **빌링키 발급 및 결제 등 정기결제 연동에 대한 자세한 내용은 <a href="https://docs.iamport.kr/implementation/subscription">정기결제 연동하기</a> 문서를 참고하세요.**|
| --- |

<Br />

PG사별 지원하는 방식 및 설정이 다르기 때문에 다음의 각 PG사별 상세 내용을 확인하세요. 두가지 방식 모두 지원하는 PG사도 있습니다.

### 결제창 방식

- [다날(카드)](./example/danal-card-request-billing-key.md)
- [다날(휴대폰)](./example/danal-phone-request-billing-key.md)
- [NHN KCP](./example/kcp-request-billing-key.md)
- [KG이니시스](./example/inicis-request-billing-key.md)
- [JTNet](./example/jtnet-request-billing-key.md)
- [KICC](./example/kicc-request-billing-key.md)
- [페이코](./example/payco-request-billing-key.md)
- [카카오페이](./example/kakaopay-request-billing-key.md)
- [차이](./example/chai-request-billing-key.md)
- [KG모빌리언스](./example/mobilians-phone-request-billing-key.md)
- [네이버페이](/NAVERPAY/sample/naverpay-recurring.md)

### REST API 방식

- [나이스페이먼츠](./example/nice-api-billing-key.md)
- [NHN KCP](./example/kcp-api-billing-key.md)
- [KG이니시스](./example/inicis-api-billing-key.md)
- [JTNet](./example/jtnet-api-billing-key.md)
- [세틀뱅크](./example/settlebank-api-billing-key.md)
- [페이조아(다우데이타)](./example/payjoa-api-billing-key.md)

