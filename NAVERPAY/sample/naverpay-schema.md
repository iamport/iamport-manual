# naverProducts JSON Schema

```javascript
{
	"$schema": "http://json-schema.org/draft-04/schema#",
	"title": "Naver Product Set",
	"description": "네이버-체크아웃 구매상품 정보",
	"type": "array",
	"items": {
		"title": "Naver Product",
		"type": "object",
		"properties": {
			"id": {
				"description": "상품고유ID",
				"type": "string"
			},
			"merchantProductId": {
				"description": "상품관리ID",
				"type": "string"
			},
			"ecMallProductId": {
				"description": "지식쇼핑 상품관리ID",
				"type": "string"
			},
			"name": {
				"description": "상품명",
				"type": "string"
			},
			"basePrice": {
				"description": "상품기본가격",
				"type": "integer"
			},
			"taxType": {
				"description": "부가세 부과 여부",
				"enum": ["TAX", "TAX_FREE"]
			},
			"quantity": {
				"description": "상품구매수량",
				"type": "integer"
			},
			"infoUrl": {
				"description": "상품상세페이지 URL",
				"type": "string"
			},
			"imageUrl": {
				"description": "상품 Thumbnail 이미지 URL",
				"type": "string"
			},
			"option": {
				"deprecated" : true,
				"description": "구매자가 선택한 상품 옵션에 대한 상세 정보",
				"$ref": "#/definitions/option"
			},
			"options": {
				"type": "array",
				"items": {
					"$ref": "#/definitions/option"
				}
			},
			"shipping": {
				"description": "상품 배송관련 상세 정보",
				"$ref": "#/definitions/shipping"
			}
		},
		"required": ["id", "name", "basePrice", "infoUrl", "imageUrl", "shipping"]
	},
	"definitions": {
		"option": {
			"properties": {
				"optionQuantity": {
					"description": "구매자가 선택한 옵션이 적용된 상품의 구매 수량",
					"type": "integer"
				},
				"optionPrice": {
					"description": "구매자가 선택한 옵션에 대한 추가 금액(-기호 허용)",
					"type": "integer"
				},
				"selectionCode": {
					"description": "구매자가 선택한 옵션 조합의 고유ID",
					"type": "string"
				},
				"selections": {
					"description": "구매자가 선택한 옵션 조합",
					"type": "array",
					"items": {
						"$ref": "#/definitions/optionItem"
					}
				}
			},
			"required": ["optionQuantity", "optionPrice"]
		},
		"optionItem": {
			"properties": {
				"code": {
					"description": "옵션 코드",
					"type": "string"
				},
				"label": {
					"description": "옵션 레이블",
					"type": "string"
				},
				"value": {
					"description": "옵션 값",
					"type": "string"
				},
			},
			"required": ["code", "label", "value"]
		},
		"shipping": {
			"properties": {
				"groupId": {
					"description": "묶음 배송을 구분하는 코드. groupId가 동일한 Naver Product들에 대해서 묶음 배송비정책이 적용됨",
					"type": "string"
				},
				"method": {
					"description": "배송방식(DELIVERY:택배·소포·등기, QUICK_SVC:퀵 서비스, DIRECT_DELIVERY:직접 전달, VISIT_RECEIPT:방문 수령, NOTHING:배송 없음)",
					"enum": ["DELIVERY", "QUICK_SVC", "DIRECT_DELIVERY", "VISIT_RECEIPT", "NOTHING"]
				},
				"baseFee": {
					"description": "기본 배송비",
					"type": "integer"
				},
				"feeType": {
					"deprecated" : true,
					"description": "배송비 적용방식(FREE:무료, CHARGE:유료, CONDITIONAL_FREE:조건부 무료, CHARGE_BY_QUANTITY:수량별 부과)",
					"enum": ["FREE", "CHARGE", "CONDITIONAL_FREE", "CHARGE_BY_QUANTITY"]
				},
				"feePayType": {
					"description": "배송비 지불방식(PREPAYED:선불, CASH_ON_DELIVERY:착불)",
					"enum": ["PREPAYED", "CASH_ON_DELIVERY"]
				},
				"feeRule": {
					"description": "조건별 배송비 규칙",
					"$ref": "#/definitions/feeRule"
				}
			},
			"required": ["baseFee", "feePayType"]
		},
		"feeRule": {
			"properties": {
				"freeByThreshold": {
					"description": "무료배송 최소 주문금액",
					"type": "integer"
				},
				"repeatByQty": {
					"description": "배송비 반복부과 기준 구매수량",
					"type": "integer"
				},
				"rangesByQty": {
					"description": "구매수량별 커스텀 추가배송비",
					"type": "array",
					"$ref": "#/definitions/feeRangeByQty"
				},
				"surchargesByArea": {
					"description": "지역별 추가배송비",
					"type": ["string", "array"],
					"$ref": "#/definitions/feeAreaByQty"
				}
			}
		},
		"feeRangeByQty": {
			"properties": {
				"from": {
					"description": "배송비 구간적용 최소 수량",
					"type": "integer"
				},
				"surcharge": {
					"description": "해당 구간에 적용되는 추가 배송비",
					"type": "integer"
				}
			}
		},
		"feeAreaByQty": {
			"properties": {
				"area": {
					"description": "지역별 배송비 구간명",
					"type": "array",
					"enum": ["island", "jeju"]
				},
				"surcharge": {
					"description": "해당 지역에 적용되는 추가 배송비",
					"type": "integer"
				}
			}
		}
	}
}
```