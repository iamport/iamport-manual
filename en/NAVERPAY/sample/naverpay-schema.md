# `naverProducts` JSON Schema

:globe_with_meridians: [KO](/NAVERPAY/sample/naverpay-schema.md)  

```javascript
{
	"$schema": "http://json-schema.org/draft-04/schema#",
	"title": "Naver Product Set",
	"description": "Naver-Checkout Product Information",
	"type": "array",
	"items": {
		"title": "Naver Product",
		"type": "object",
		"properties": {
			"id": {
				"description": "Product ID",
				"type": "string"
			},
			"merchantProductId": {
				"description": "Merchant managed Product ID",
				"type": "string"
			},
			"ecMallProductId": {
				"description": "Knowledge shopping Product ID",
				"type": "string"
			},
			"name": {
				"description": "Product name",
				"type": "string"
			},
			"basePrice": {
				"description": "Product base price",
				"type": "integer"
			},
			"taxType": {
				"description": "Option to tax",
				"enum": ["TAX", "TAX_FREE"]
			},
			"quantity": {
				"description": "Purchase quantity",
				"type": "integer"
			},
			"infoUrl": {
				"description": "Product page URL",
				"type": "string"
			},
			"imageUrl": {
				"description": "Product thumbnail image URL",
				"type": "string"
			},
			"option": {
				"deprecated" : true,
				"description": "Details about selected options",
				"$ref": "#/definitions/option"
			},
			"options": {
				"type": "array",
				"items": {
					"$ref": "#/definitions/option"
				}
			},
			"shipping": {
				"description": "Shipping details",
				"$ref": "#/definitions/shipping"
			}
		},
		"required": ["id", "name", "basePrice", "infoUrl", "imageUrl", "shipping"]
	},
	"definitions": {
		"option": {
			"properties": {
				"optionQuantity": {
					"description": "Number of items with the selected option",
					"type": "integer"
				},
				"optionPrice": {
					"description": "Amount charged for the selected option (may be negative)",
					"type": "integer"
				},
				"selectionCode": {
					"description": "ID for the option combination",
					"type": "string"
				},
				"selections": {
					"description": "Selected option combination",
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
					"description": "Option code",
					"type": "string"
				},
				"label": {
					"description": "Option label",
					"type": "string"
				},
				"value": {
					"description": "Option value",
					"type": "string"
				},
			},
			"required": ["code", "label", "value"]
		},
		"shipping": {
			"properties": {
				"groupId": {
					"description": "Shipping group ID. Products with the same groupId are shipped together.",
					"type": "string"
				},
				"method": {
					"description": "Shipping method (DELIVERY (Courier Delivery/Parcel/Registered), QUICK_SVC (Quick Service), DIRECT_DELIVERY (Direct Delivery), VISIT_RECEIPT (Private Pick-up), NOTHING (No Delivery)",
					"enum": ["DELIVERY", "QUICK_SVC", "DIRECT_DELIVERY", "VISIT_RECEIPT", "NOTHING"]
				},
				"baseFee": {
					"description": "Base shipping rate",
					"type": "integer"
				},
				"feeType": {
					"deprecated" : true,
					"description": "Shipping rate type (FREE: free, CHARGE: paid, CONDITIONAL_FREE: conditional free shipping, CHARGE_BY_QUANTITY: charged by quantity)",
					"enum": ["FREE", "CHARGE", "CONDITIONAL_FREE", "CHARGE_BY_QUANTITY"]
				},
				"feePayType": {
					"description": "Shipping pay method (PREPAYED: prepaid, CASH_ON_DELIVERY: cash payment on delivery)",
					"enum": ["PREPAYED", "CASH_ON_DELIVERY"]
				},
				"feeRule": {
					"description": "Shipping rate policy",
					"$ref": "#/definitions/feeRule"
				}
			},
			"required": ["baseFee", "feePayType"]
		},
		"feeRule": {
			"properties": {
				"freeByThreshold": {
					"description": "Free shipping threshold",
					"type": "integer"
				},
				"repeatByQty": {
					"description": "Number of items per which base fee is charged",
					"type": "integer"
				},
				"rangesByQty": {
					"description": "Array of objects that define additional shipping fee for custom item quantity range",
					"type": "array",
					"$ref": "#/definitions/feeRangeByQty"
				},
				"surchargesByArea": {
					"description": "Array of objects that define extended area surcharge",
					"type": ["string", "array"],
					"$ref": "#/definitions/feeAreaByQty"
				}
			}
		},
		"feeRangeByQty": {
			"properties": {
				"from": {
					"description": "Minimum quantity of this range",
					"type": "integer"
				},
				"surcharge": {
					"description": "Additional shipping fee applied to this quantity range",
					"type": "integer"
				}
			}
		},
		"feeAreaByQty": {
			"properties": {
				"area": {
					"description": "Area name",
					"type": "array",
					"enum": ["island", "jeju"]
				},
				"surcharge": {
					"description": "Surcharge applied to this area",
					"type": "integer"
				}
			}
		}
	}
}
```