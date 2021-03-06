# API Specification for ZeekSolutions - Circle K VN API Integration (Phase 5)

# Changelog

Date|	Author|	Content
---|---|---
2021-08-02|	Zeek team|Initial draft

# Introduction

This document includes the APIs for Phase 5 integration, which is mainly the interaction between CKGo (ZeekSolutions) and NRI / Circle K Central Server.

# General

## ref_xxx_id parameters

When you come across `ref_xxx_id` parameter, it is referring to the ID in client's system, ie. CKVN system.

## Store

In ZeekSolutions (CKGo), multiple stores (ie. Brand) are supported. At this moment, there is only one store for CKVN. Please get Store ID from Zeek team if needed.

## Location

"Location" means restaurant / branch. There are multiple CKVN locations in the city. 

In ZeekSolutions, each location can sell different products. Therefore, you need to specify the location IDs when you update product. 

## Inventory

ZeekSolutions support individual inventory number of a product in each location.

## Combo items

In ZeekSolutions, product supports Combo or a la carte. For a la carte products, they support inventory number. However, there is no inventory number for Combo product. The stock status of Combo product is deduced from its sub-items. 

## Product options

Merchant can set options for products, like "Sweetness level" for drinks. Options apply to both Combo and a la carte products. However, options do not count stock.

# API Authentication

## CKVN requests Zeek API

You needs to include access token in each API calls. You can obtain the access token by GetAccessToken API. 

Details will be provided later.

## Zeek requests CKVN API

When Zeek requests CKVN API, a HMAC-SHA256 generated signature will be included. CKVN can verify the request by this signature. 

Details will be provided later.

# API Endpoints

Here are API endpoints:

| API| Usage
|---|---|
|[GetAccessToken](#getaccesstoken-get-access-token)|Get access token. You needs to include access token in each API calls.
|[CreateProduct](#createproduct-create-a-product)|CKVN creates product to Zeek. It should be used in delta update of product list.
|[BatchUpdateProduct](#batchupdateproduct-batch-update-product)|CKVN update multiple products to Zeek.
|[OrderCreateCallback](#ordercreatecallback-order-create-callback)|When an order is created in Zeek, this callback API will be called to send order details to CKVN
|[CheckProviderInventory](#checkproviderinventory-check-provider-inventory)|When customer is going to checkout, Zeek will call this API (provided by CKVN) to check whether the items are in stock.

### Authentication

#### [GetAccessToken] Get access token

> HTTP POST /api/v1.0/auth/token

|||
|---|---|
|**API Type**| Platform API
|**Usage**|Get the access token for API authentication

##### Request

Name | Type | Required | Default | Description | Remarks
---|---|---|---|---|---
client_id|string|Y|-| App Key|
client_secret|string|Y|-| App Secret|
grant_type|string|N|`client_credentials`| OAuth grant type|Support "client_credentials" only
scope|string|N|`all`| OAuth scope|Support "all" only



Example???

```json
{
  "client_id": "bxF8hHT9",
  "client_secret": "Zp69S8f0UdmDIB11",
  "grant_type": "client_credentials",
  "scope": "all"
}
```

##### Response


**HTTP 200 :** Success


Name | Type | Description | Remarks
---|---|---|---
token_type|string|Token type|Need to include in HTTP Authorization Header (Support Bearer only)
access_token|string|Access token|
expires_in|int|TTL|Unit: second
scope|string|Scope|Support "all" only.



Example???

```json
{
  "error" : 0,
  "data": {
    "token_type": "Bearer",
    "access_token": "xxxxxxxxxxxxxxxxxxxxxxx",
    "expires_in": 86400,
    "scope": "all"
  }
}
```


**HTTP 400 :** Invalid parameters


Example???

```json
{
  "error": "40001",
  "error_msg": "Invalid parameter"
}
```


**HTTP 400 :** Invalid App Key or App Secret


Example???

```json
{
  "error": "40002",
  "error_msg": "Invalid App Key or App Secret"
}
```

### Product

#### [CreateProduct] Create a product

> HTTP POST /api/v1.0/product/product/create

|||
|---|---|
|**API Type**| Platform API
|**Usage**|Create a product

##### Request

Name | Type | Required | Default | Description | Remarks
---|---|---|---|---|---
ref_product_id|string|N|| Ref ID|
name|object|Y|| Product name|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
name.en|string|N|| Product name???English???|
name.zh-HK|string|N|| Product name???Chinese???|
description|object|N|| Product description|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
description.en|string|N|| Product description???English???|
description.zh-HK|string|N|| Product description???Chinese???|
type|int|Y|| Product type|- 1 : ?? la carte<br />- 2 : Combo
store|object|Y|`0`| Store info|
store.store_id|int|Y|| Store ID|
locations|array|N|`Array`| Locations|- Each product can be sold in multiple locations. You need to specific the location IDs here.<br />- If no location is input, that means this product will not be shown in any location.
locations[].location_id|int|N|| Location ID|
categories|array|N|`Array`| Categories|If no category is input, that means this product will not be shown in any category.
categories[].category_id|int|N|| Category ID|
sku|string|Y|| SKU|
model|string|Y|| Product Model|
sort_order|int|N|| Sort order|In descending order
price|object|Y|| Price|
price.amount|string|Y|| Amount|
discount_price|array|N|| Bulk purchase discount price|
discount_price[].price|object|N|| Price|
discount_price[].price.amount|string|N|| Amount|
discount_price[].purchase_count|int|N|| Purchase count|
discount_price[].sort_order|int|N|| Sort order|In descending order
discount_price[].start_date|string|N|| Start date|Format: YYYY-MM-DD
discount_price[].end_date|string|N|| End date|Format???YYYY-MM-DD
special_price|array|N|| Special price|
special_price[].price|object|N|| Price|
special_price[].price.amount|string|N|| Amount|
special_price[].sort_order|int|N|| Sort order|In descending order
special_price[].start_date|string|N|| Start date|Format: YYYY-MM-DD
special_price[].end_date|string|N|| End date|Format???YYYY-MM-DD
is_deduct_stock|int|Y|| Deduct stock?|- 0 : No<br />- 1 : Yes
is_individual_sale|int|Y|| Can sell individually|- 0 : No<br />- 1 : Yes
is_alcohol|int|Y|| Is alcohol product|- 0 : No<br />- 1 : Yes
launch|object|Y|| Launch settings|
launch.status|int|Y|| Status|- `1`: Launched<br />- `0` : Not launched
launch.start_time|string|N|| Start time|If it is not input, that means there is no constraint on the start time
launch.end_time|string|N|| End time|If it is not input, that means there is no constraint on the end time
timeslots|array|Y|| Product available timeslots|
timeslots[].timeslot_id|int|Y|| Timeslot ID|
payment_methods|array|Y|| Payment methods|It it is empty, that means no payment method is supported for this product.
payment_methods[].key|string|N|| Payment method key|The payment method key is used to identify payment method in the platform.
departments|array|N|| Departments|
departments[].department_id|int|N|| Department ID|
combo_components|array|N|`Array`| Combo components|
combo_components[].component_id|int|N|| Component ID|
combo_components[].name|object|N|| Component name|
combo_components[].name.en|string|N|| Component name???English???|
combo_components[].name.zh-HK|string|N|| Component name???Chinese???|
combo_components[].item_select_quantity|int|N|| The quantity to select items|
combo_components[].is_required|int|N|| Is required?|- 0 : Optional<br />- 1 : Mandatory
combo_components[].items|array|N|`Array`| Component items|
combo_components[].items[].product_id|int|N|| Product ID|
combo_components[].items[].price|object|N|| Price|
combo_components[].items[].price.amount|string|Y|| Amount|
combo_components[].items[].price_adjustment|int|Y|| Adjust method|- 1 : Add<br />- 2 : Deduct
combo_components[].items[].sort_order|int|N|| Sort order|In descending order
options|array|N|| Product options|
options[].option_id|int|N|| Product option ID|
options[].is_required|int|N|| Is required?|- 0 : Optional<br />- 1 : Mandatory
options[].option_values|array|N|`0`| Option values|
options[].option_values[].value_id|int|N|| Option value ID|
options[].option_values[].price|object|N|| Price|
options[].option_values[].price.amount|string|Y|| Amount|
options[].option_values[].price_adjustment|int|Y|| Adjust method|- 1 : Add<br />- 2 : Deduct
purchase_quantity|object|Y|| Purchase quantity settings|
purchase_quantity.minimum|int|Y|| Minimum|
purchase_quantity.maximum|int|Y|| Maximum|
spec|object|N|| Specification|
spec.weight|float|N|| Weight|
meta|object|N|| Meta|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
meta.title|object|N|| Meta title|
meta.title.en|string|N|| Meta title???English???|
meta.title.zh-HK|string|N|| Meta title???Chinese???|
meta.description|object|N|| Meta description|
meta.description.en|string|N|| Meta description???English???|
meta.description.zh-HK|string|N|| Meta description???Chinese???|
meta.keyword|object|N|| Meta keyword|
meta.keyword.en|string|N|| Meta keyword???English???|
meta.keyword.zh-HK|string|N|| Meta keyword???Chinese???|



Example???

```json
{
  "error" : 0,
  "data": {
    "ref_product_id": "product-1",
    "name": {
      "en": "Coca-cola",
      "zh-HK": "????????????"
    },
    "description": {
      "en": "Coca-cola 350ml",
      "zh-HK": "???????????? 350ml"
    },
    "type": 1,
    "store": {
      "store_id": 145
    },
    "locations": [
      {
        "location_id": 33
      }
    ],
    "categories": [
      {
        "category_id": 100
      }
    ],
    "sku": "coca-cola-sku",
    "model": "coca-cola-model",
    "sort_order": 13,
    "price": {
      "amount": "45.40"
    },
    "discount_price": [
      {
        "price": {
          "amount": "45.40"
        },
        "purchase_quantity": 300,
        "sort_order": 15,
        "start_date": "2020-03-03",
        "end_date": "2020-03-10"
      }
    ],
    "special_price": [
      {
        "price": {
          "amount": "45.40"
        },
        "sort_order": 15,
        "start_date": "2020-03-03",
        "end_date": "2020-03-10"
      }
    ],
    "is_deduct_stock": 1,
    "is_individual_sale": 1,
    "is_alcohol": 1,
    "launch": {
      "status": 1,
      "start_time": "2004-05-03T17:30:08+07:00",
      "end_time": "2004-05-03T17:30:08+07:00"
    },
    "timeslots": [
      {
        "timeslot_id": 67
      }
    ],
    "payment_methods": [
      {
        "key": "stripe"
      },
      {
        "key": "paypal"
      }
    ],
    "departments": [
      {
        "department_id": 67
      }
    ],
    "combo_components": [
      {
        "component_id": 64,
        "name": {
          "en": "Drinks",
          "zh-HK": "??????"
        },
        "is_required": 1,
        "item_select_quantity": 12,
        "items": [
          {
            "product_id": 33,
            "price": {
              "amount": 300
            },
            "price_adjustment": 1,
            "sort_order": 100
          }
        ]
      }
    ],
    "options": [
      {
        "option_id": 64,
        "is_required": 1,
        "option_values": [
          {
            "value_id": 330,
            "price": {
              "amount": 300
            },
            "price_adjustment": 1
          }
        ]
      }
    ],
    "purchase_quantity": {
      "minimum": 0,
      "maximum": 10
    },
    "spec": {
      "weight": 30.3
    },
    "meta": {
      "title": {
        "en": "Coca-cola",
        "zh-HK": "????????????"
      },
      "description": {
        "en": "Coca-cola",
        "zh-HK": "????????????"
      },
      "keyword": {
        "en": "Coca-cola",
        "zh-HK": "????????????"
      }
    }
  }
}
```

##### Response


**HTTP 200 :** Success


Name | Type | Description | Remarks
---|---|---|---
product_id|int|Product ID|



Example???

```json
{
  "error" : 0,
  "data": {
    "product_id" : 200
  }
}
```


**HTTP 400 :** Invalid parameters


Example???

```json
{
  "error": "40001",
  "error_msg": "Invalid parameter"
}
```


**HTTP 401 :** Unauthorised


Example???

```json
{
  "error": "40101",
  "error_msg": "Unauthorised"
}
```

#### [BatchUpdateProduct] Batch update product

> HTTP POST /api/v1.0/product/product/batch_update

|||
|---|---|
|**API Type**| Platform API
|**Usage**|Update product in batch

##### Request

Name | Type | Required | Default | Description | Remarks
---|---|---|---|---|---
products|array|Y|-| Products|
products[].product_id|int|Y|| Product ID|
products[].name|object|Y|| Product name|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
products[].name.en|string|N|| Product name???English???|
products[].name.zh-HK|string|N|| Product name???Chinese???|
products[].description|object|Y|| Product description|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
products[].description.en|string|N|| Product description???English???|
products[].description.zh-HK|string|N|| Product description???Chinese???|
products[].type|int|Y|| Product type|- 1 : ?? la carte<br />- 2 : Combo
products[].locations|array|N|`Array`| Locations|- Each product can be sold in multiple locations. You need to specific the location IDs here.<br />- If no location is input, that means this product will not be shown in any location.
products[].locations[].location_id|int|N|| Location ID|
products[].categories|array|Y|`Array`| Categories|If no category is input, that means this product will not be shown in any category.
products[].categories[].category_id|int|N|| Category ID|
products[].sort_order|int|N|| Sort order|In descending order
products[].price|object|Y|| Price|
products[].price.amount|string|Y|| Amount|
products[].special_price|array|N|| Special price|
products[].special_price[].price|object|N|| Price|
products[].special_price[].price.amount|string|N|| Amount|
products[].special_price[].sort_order|int|N|| Sort order|In descending order
products[].special_price[].start_date|string|N|| Start date|Format: YYYY-MM-DD
products[].special_price[].end_date|string|N|| End date|Format???YYYY-MM-DD
products[].is_individual_sale|int|Y|| Can sell individually|- 0 : No<br />- 1 : Yes
products[].is_alcohol|int|Y|| Is alcohol product|- 0 : No<br />- 1 : Yes
products[].launch|object|Y|| Launch settings|
products[].launch.status|int|Y|| Status|- `1`: Launched<br />- `0` : Not launched
products[].launch.start_time|string|N|| Start time|If it is not input, that means there is no constraint on the start time
products[].launch.end_time|string|N|| End time|If it is not input, that means there is no constraint on the end time
products[].timeslots|array|Y|| Product available timeslots|
products[].timeslots[].timeslot_id|int|Y|| Timeslot ID|
products[].combo_components|array|N|`Array`| Combo components|
products[].combo_components[].component_id|int|N|| Component ID|
products[].combo_components[].name|object|N|| Component name|
products[].combo_components[].name.en|string|N|| Component name???English???|
products[].combo_components[].name.zh-HK|string|N|| Component name???Chinese???|
products[].combo_components[].item_select_quantity|int|N|| The quantity to select items|
products[].combo_components[].is_required|int|N|| Is required?|- 0 : Optional<br />- 1 : Mandatory
products[].combo_components[].items|array|N|`Array`| Component items|
products[].combo_components[].items[].product_id|int|N|| Product ID|
products[].combo_components[].items[].price|object|N|| Price|
products[].combo_components[].items[].price.amount|string|Y|| Amount|
products[].combo_components[].items[].price_adjustment|int|Y|| Adjust method|- 1 : Add<br />- 2 : Deduct
products[].combo_components[].items[].sort_order|int|N|| Sort order|In descending order
products[].options|array|N|| Product options|
products[].options[].option_id|int|N|| Product option ID|
products[].options[].is_required|int|N|| Is required?|- 0 : Optional<br />- 1 : Mandatory
products[].options[].option_values|array|N|`0`| Option values|
products[].options[].option_values[].value_id|int|N|| Option value ID|
products[].options[].option_values[].price|object|N|| Price|
products[].options[].option_values[].price.amount|string|Y|| Amount|
products[].options[].option_values[].price_adjustment|int|Y|| Adjust method|- 1 : Add<br />- 2 : Deduct
products[].purchase_quantity|object|Y|| Purchase quantity settings|
products[].purchase_quantity.minimum|int|Y|| Minimum|
products[].purchase_quantity.maximum|int|Y|| Maximum|
products[].spec|object|N|| Specification|
products[].spec.weight|float|N|| Weight|
products[].meta|object|N|| Meta|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
products[].meta.title|object|N|| Meta title|
products[].meta.title.en|string|N|| Meta title???English???|
products[].meta.title.zh-HK|string|N|| Meta title???Chinese???|
products[].meta.description|object|N|| Meta description|
products[].meta.description.en|string|N|| Meta description???English???|
products[].meta.description.zh-HK|string|N|| Meta description???Chinese???|
products[].meta.keyword|object|N|| Meta keyword|
products[].meta.keyword.en|string|N|| Meta keyword???English???|
products[].meta.keyword.zh-HK|string|N|| Meta keyword???Chinese???|



Example???

```json
{
  "products": [
    {
      "product_id": 78,
      "name": {
        "en": "Coca-cola",
        "zh-HK": "????????????"
      },
      "description": {
        "en": "Coca-cola 350ml",
        "zh-HK": "???????????? 350ml"
      },
      "type": 1,
      "locations": [
        {
          "location_id": 44
        }
      ],
      "categories": [
        {
          "category_id": 100
        }
      ],
      "sort_order": 13,
      "price": {
        "amount": "45.40"
      },
      "special_price": [
        {
          "price": {
            "amount": "45.40"
          },
          "sort_order": 15,
          "start_date": "2020-03-03",
          "end_date": "2020-03-10"
        }
      ],
      "is_individual_sale": 1,
      "is_alcohol": 1,
      "launch": {
        "status": 1,
        "start_time": "2004-05-03T17:30:08+07:00",
        "end_time": "2004-05-03T17:30:08+07:00"
      },
      "timeslots": [
        {
          "timeslot_id": 67
        }
      ],
      "combo_components": [
        {
          "component_id": 64,
          "name": {
            "en": "Drinks",
            "zh-HK": "??????"
          },
          "is_required": 1,
          "item_select_quantity": 12,
          "items": [
            {
              "product_id": 33,
              "price": {
                "amount": 300
              },
              "price_adjustment": 1,
              "sort_order": 100
            }
          ]
        }
      ],
      "options": [
        {
          "option_id": 64,
          "is_required": 1,
          "option_values": [
            {
              "value_id": 330,
              "price": {
                "amount": 300
              },
              "price_adjustment": 1
            }
          ]
        }
      ],
      "purchase_quantity": {
        "minimum": 0,
        "maximum": 10
      },
      "spec": {
        "weight": 30.3
      },
      "meta": {
        "title": {
          "en": "Coca-cola",
          "zh-HK": "????????????"
        },
        "description": {
          "en": "Coca-cola",
          "zh-HK": "????????????"
        },
        "keyword": {
          "en": "Coca-cola",
          "zh-HK": "????????????"
        }
      }
    }
  ]
}
```

##### Response


**HTTP 200 :** Success


Name | Type | Description | Remarks
---|---|---|---
results|array|Results|
results[].item_id|int|Inventory item ID|
results[].action_result|int|Action result|- 0 : Failed<br />- 1 : Success
results[].error_msg|string|Error message|



Example???

```json
{
  "error" : 0,
  "data": {
    "results": [
      {
        "product_id": 34,
        "action_result": 1,
        "error_msg": ""
      },
      {
        "product_id": 35,
        "action_result": 0,
        "error_msg": "No such item"
      }
    ]
  }
}
```


**HTTP 400 :** Invalid parameters


Example???

```json
{
  "error": "40001",
  "error_msg": "Invalid parameter"
}
```


**HTTP 401 :** Unauthorised


Example???

```json
{
  "error": "40101",
  "error_msg": "Unauthorised"
}
```


**HTTP 404 :** Not found


Example???

```json
{
  "error": "40401",
  "error_msg": "Not found"
}
```

### Order

#### [OrderCreateCallback] Order create callback

> HTTP POST [The URL is provided by the Client]

|||
|---|---|
|**API Type**| Webhook API
|**Usage**|- When an order is created, the order details will be sent to the client system by this callback.<br />- In this API, `event = "order_create"`

##### Request

Name | Type | Required | Default | Description | Remarks
---|---|---|---|---|---
event|string|Y|| Callback event|In each callback API, event will be included to specify the event it is going to be sent from Zeek Platform.
payload|object|Y|| Event payload|
payload.order_id|string|Y|| Order ID|
payload.order_time|string|Y|| Order creation time|
payload.pickup_code|string|Y|| Pickup code|
payload.store|object|Y|| Store|
payload.store.store_id|int|Y|| Store ID|
payload.location|object|Y|| Location|
payload.location.location_id|int|Y|| Location ID|
payload.payment|object|Y|| Payment setting|
payload.payment.method|string|Y|| Payment method|
payload.schedule|object|Y|| Order schedule setting|
payload.schedule.type|int|Y|| Schedule type|- 1: Realtime order<br />- 2: Advanced order
payload.schedule.delivery_time|string|Y|| Scheduled delivery time|- The scheduled time to deliver products to customer.<br />- If it is delivery order, it will be the time when the partner (rider) delivers to customer. <br />- If it is pickup order, it will be the time when customer comes to store to pickup.<br />- If it is realtime order, this parameter will be `null`.
payload.shipping|object|Y|| Shipping setting|
payload.shipping.type|int|Y|| Shipping type|- 1: Customer pickup<br />- 2: Delivery
payload.remark|string|Y|| Order remark|
payload.fee|object|Y|| Order fee|
payload.fee.subtotal|object|Y|| Subtotal|
payload.fee.subtotal.currency_code|string|Y|| Currency|
payload.fee.subtotal.amount|string|Y|| Amount|
payload.fee.minimum_purchase|object|Y|| Minimum purchase|
payload.fee.minimum_purchase.currency_code|string|Y|| Currency|
payload.fee.minimum_purchase.amount|string|Y|| Amount|
payload.fee.difference_to_minimum|object|Y|| Difference to minimum|
payload.fee.difference_to_minimum.currency_code|string|Y|| Currency|
payload.fee.difference_to_minimum.amount|string|Y|| Amount|
payload.fee.coupon_discount|object|Y|| Coupon discount|
payload.fee.coupon_discount.currency_code|string|Y|| Currency|
payload.fee.coupon_discount.amount|string|Y|| Amount|
payload.fee.shipping_fee|object|Y|| Shipping fee|
payload.fee.shipping_fee.currency_code|string|Y|| Currency|
payload.fee.shipping_fee.amount|string|Y|| Amount|
payload.fee.vat|object|Y|| VAT|
payload.fee.vat.currency_code|string|Y|| Currency|
payload.fee.vat.amount|string|Y|| Amount|
payload.fee.is_vat_included|int|Y|| Is VAT included?|- 0 : Not included<br />- 1 : Included
payload.fee.total|object|Y|| Order total|
payload.fee.total.currency_code|string|Y|| Currency|
payload.fee.total.amount|string|Y|| Amount|
payload.customer|object|Y|| Order fee|
payload.customer.member_id|string|Y|| Member ID|
payload.customer.ref_member_id|string|Y|| Client Member ID|The member ID from Client system
payload.customer.name|string|Y|| Customer name|
payload.customer.phone|object|Y|| Phone|
payload.customer.phone.country_code|string|Y|| Country code|
payload.customer.phone.phone_number|string|Y|| Phone number|
payload.customer.address|object|Y|| Address|
payload.customer.address.address_text|string|Y|| Address text|
payload.products|array|Y|| Order items|
payload.products[].product_id|int|N|| Product ID|
payload.products[].ref_product_id|string|N|| Ref ID|
payload.products[].name|object|N|| Product name|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
payload.products[].name.en|string|N|| Product name???English???|
payload.products[].name.zh-HK|string|N|| Product name???Chinese???|
payload.products[].type|int|N|| Product type|- 1 : ?? la carte<br />- 2 : Combo
payload.products[].quantity|int|N|| Quantity|
payload.products[].price|object|N|| Price|
payload.products[].price.currency_code|string|Y|| Currency|
payload.products[].price.amount|string|Y|| Amount|
payload.products[].sale_discount_price|array|N|| Sale discount price|The discount after calculation from different settings
payload.products[].is_alcohol|int|N|| Is alcohol product|- 0 : No<br />- 1 : Yes
payload.products[].combo_components|array|N|`Array`| Combo components|
payload.products[].combo_components[].component_id|int|N|| Component ID|
payload.products[].combo_components[].name|object|N|| Component name|
payload.products[].combo_components[].name.en|string|N|| Component name???English???|
payload.products[].combo_components[].name.zh-HK|string|N|| Component name???Chinese???|
payload.products[].combo_components[].items|array|N|`Array`| Component items|
payload.products[].combo_components[].items[].product_id|int|N|| Product ID|
payload.products[].combo_components[].items[].ref_product_id|string|N|| Ref ID|
payload.products[].combo_components[].items[].name|object|N|| Product name|Please input the languages in your region. <br />- Hong Kong: `zh-HK` (Trad. Chinese), `en` (English)<br />- Bangkok: `th` (Thai), `en` (English)<br />- Singapore: `en` (English)<br />- Ho Chi Minh City: `vi` (Vietnamese), `en` (English)<br /><br />Example for Ho Chi Minh City:<br />```{"en": "Distilled water","vi": "N?????c c???t",}```
payload.products[].combo_components[].items[].name.en|string|N|| Product name???English???|
payload.products[].combo_components[].items[].name.zh-HK|string|N|| Product name???Chinese???|
payload.products[].combo_components[].items[].quantity|int|N|| Quantity|
payload.products[].combo_components[].items[].price|object|N|| Price|
payload.products[].combo_components[].items[].price.amount|string|Y|| Amount|
payload.products[].combo_components[].items[].price_adjustment|int|Y|| Adjust method|- 1 : Add<br />- 2 : Deduct
payload.products[].options|array|N|| Product options|
payload.products[].options[].option_id|int|N|| Product option ID|
payload.products[].options[].name|object|N|| Product option name|
payload.products[].options[].name.en|string|N|| Product option name???English???|
payload.products[].options[].name.zh-HK|string|N|| Product option name???Chinese???|
payload.products[].options[].option_values|array|N|`0`| Option values|
payload.products[].options[].option_values[].value_id|int|N|| Option value ID|
payload.products[].options[].option_values[].name|object|N|| Option value name|
payload.products[].options[].option_values[].name.en|string|N|| Option value name???English???|
payload.products[].options[].option_values[].name.zh-HK|string|N|| Option value name???Chinese???|
payload.products[].options[].option_values[].price|object|N|| Price|
payload.products[].options[].option_values[].price.currency_code|string|Y|| Currency|
payload.products[].options[].option_values[].price.amount|string|Y|| Amount|
payload.products[].options[].option_values[].price_adjustment|int|Y|| Adjust method|- 1 : Add<br />- 2 : Deduct



Example???

```json
{
  "event": "order_create",
  "payload": {
    "order_id": "202011991199",
    "order_time": "2004-05-03T17:30:08+07:00",
    "pickup_code": "1234",
    "store": {
      "store_id": 100
    },
    "location": {
      "location_id": 122
    },
    "payment": {
      "method": "paypal"
    },
    "schedule": {
      "type": 2,
      "delivery_time": "2004-05-03T17:30:08+07:00"
    },
    "shipping": {
      "type": 1
    },
    "remark": "Please include utensils",
    "fee": {
      "subtotal": {
        "currency_code": "HKD",
        "amount": "10.22"
      },
      "minimum_purchase": {
        "currency_code": "HKD",
        "amount": "10.22"
      },
      "difference_to_minimum": {
        "currency_code": "HKD",
        "amount": "10.22"
      },
      "coupon_discount": {
        "currency_code": "HKD",
        "amount": "10.22"
      },
      "shipping_fee": {
        "currency_code": "HKD",
        "amount": "10.22"
      },
      "vat": {
        "currency_code": "HKD",
        "amount": "10.22"
      },
      "is_vat_included": true,
      "total": {
        "currency_code": "HKD",
        "amount": "10.22"
      }
    },
    "customer": {
      "member_id": "33333",
      "ref_member_id": "ref-333333",
      "name": "Joseph Tai",
      "phone": {
        "country_code": "852",
        "phone_number": "98989898"
      },
      "address": {
        "address_text": "11/F, SF Centre, 36 Tsing Yi Hong Wan Road"
      }
    },
    "products": [
      {
        "product_id": 13,
        "ref_product_id": "ref-13",
        "name": {
          "en": "Breakfast Set A",
          "zh-HK": "?????? A"
        },
        "type": 2,
        "quantity": 3,
        "price": {
          "currency_code": "HKD",
          "amount": "45.40"
        },
        "sale_discount_price": {
          "currency_code": "HKD",
          "amount": "45.40"
        },
        "is_alcohol": 1,
        "combo_components": [
          {
            "component_id": 64,
            "name": {
              "en": "Drinks",
              "zh-HK": "??????"
            },
            "items": [
              {
                "product_id": 14,
                "ref_product_id": "ref-14",
                "name": {
                  "en": "Coffee",
                  "zh-HK": "??????"
                },
                "price": {
                  "currency_code": "HKD",
                  "amount": "45.40"
                },
                "price_adjustment": 1,
                "quantity": 3
              }
            ]
          }
        ],
        "options": [
          {
            "option_id": 64,
            "name": {
              "en": "Sweetness",
              "zh-HK": "??????"
            },
            "option_values": [
              {
                "value_id": 69,
                "name": {
                  "en": "No sugar",
                  "zh-HK": "??????"
                },
                "price": {
                  "currency_code": "HKD",
                  "amount": "45.40"
                },
                "price_adjustment": 1
              }
            ]
          }
        ]
      }
    ]
  }
}
```

##### Response


**HTTP 200 :** Success


Name | Type | Description | Remarks
---|---|---|---
status|string|Callback status|Please return `"OK"` when the callback data is received successfully.



Example???

```json
{
  "status": "OK"
}
```


### Cart

#### [CheckProviderInventory] Check Provider Inventory

> HTTP POST [The URL is provided by the Client]

|||
|---|---|
|**API Type**| Provider API
|**Usage**|- When customer is going to place order, Zeek platform will make request to Provider's API, check whether it has sufficient inventory.

##### Request

Name | Type | Required | Default | Description | Remarks
---|---|---|---|---|---
store|object|Y|| Store|
store.store_id|int|Y|| Store ID|
location|object|Y|| Location|
location.location_id|int|Y|| Location ID|
schedule|object|Y|| Order schedule setting|
schedule.type|int|Y|| Schedule type|- 1: Realtime order<br />- 2: Advanced order
schedule.delivery_time|string|Y|| Scheduled delivery time|- The scheduled time to deliver products to customer.<br />- If it is delivery order, it will be the time when the partner (rider) delivers to customer. <br />- If it is pickup order, it will be the time when customer comes to store to pickup.<br />- If it is realtime order, this parameter will be `null`.
products|array|Y|| Order items|
products[].product_id|int|Y|| Product ID|
products[].ref_product_id|string|Y|| Ref ID|
products[].quantity|int|Y|| Quantity|



Example???

```json
{
  "store": {
    "store_id": 100
  },
  "location": {
    "location_id": 122
  },
  "schedule": {
    "type": 2,
    "delivery_time": "2004-05-03T17:30:08+07:00"
  },
  "shipping": {
    "type": 1
  },
  "products": [
    {
      "product_id": 13,
      "ref_product_id": "ref-13",
      "quantity": 3
    }
  ]
}
```

##### Response


**HTTP 200 :** Success


Name | Type | Description | Remarks
---|---|---|---
products|array|Order items|
products[].product_id|int|Product ID|
products[].inventory_status|int|Inventory status|- 1: In stock<br />- 2: Out of stock



Example???

```json
{
  "products": [
    {
      "product_id": 33,
      "inventory_status": 1
    }
  ]
}
```
