---
title: bookkeep.com API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the documentation for bookkeep.com's accounting integration platform.  

We have language bindings in Shell, Ruby, Python, and JavaScript! You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

This example API documentation page was created with [Slate](https://github.com/slatedocs/slate). Feel free to edit it and use it as a base for your own API's documentation.

# Summary Composition

To create a daily summary for a revenue cloud app you will need to understand some concepts.

## Which orders do we summarize?

We are doing accrual accounting which means that we summarize for the day the payment was made against the order.  Often this required pulling the payments first and then backing into the orders.  Or pulling orders which are completed or paid.  

In addition returns processed on the summary date are also included so finding financial transactions that were money returned will then surface the retur order depending on the system.  

## Sales Tax Node
```javascript
  { 
        "rate": 0.08875,
        "label": "US-NY-Sales Tax 8.875%-8.87500",
        "amount": 226.85,
        "taxablesales": 2556.06
      }
```

Sales tax is broken down in the `subcategories` node as follows:

* `rate` is decimal format out to 5 digits and comes from the source system
* `label` should follow this format `{2 char country code}-{2 char province/state code}-{Name Here}-{rate as percentage to 5 digits}` where the first 2 characters are the country code, the next 2 are a province code and the next is the name from the source system and finally the rate as a percentage.  
* `amount` is the actual amount from the source system grouped by the label above.
* `taxablesales` is calculated as `amount  / rate` and should be rounded to 2 digits. 




```javascript
 {
  "inputs": {
    "testing": false,
    "journal_entry_template": "my_summary"
  },
  "je_date": "2021-01-29",
  "je_type": "my_summary",
  "currency": "USD",
  "realm_id": "9130348211837726",
  "docnumber": "BKShopSum2901214584",
  "api_version": "2020-02-04",
  "build_number": "0.1.383",
  "bk_external_id": 4584,
  "source_report_url": "my-source-system.com/comapre-numbers-to-this-report.html",
  "je_private_note": "note to pass through to accounting system",
  "gross_sales": {
    "amount": 0,
    "subcategories": []
  },
   "shipping_income": {
    "amount": 0
  },
  "discounts": {
    "amount": 0,
    "subcategories": []
  },
  "returns": {
    "amount": 0
  },
  "gift_cards_issued": {
    "amount": 0
  },
  "gratuity_collected": {
    "amount": 0
  },
  "sales_tax_collected": {
    "amount": 0,
    "subcategories": []
  },
  "manual_payments": {
    "amount": 0,
    "subcategories": []
  },
  "paypal_payments": {
    "amount": 0
  },
  "shopify_payments": {
    "amount": 0,
    "subcategories": [
      {
        "label": "Today Batch",
        "amount": 0
      },
      {
        "label": "Tomorrow Batch",
        "amount": 0
      }
    ]
  },
  "unknown_payments": {
    "amount": 0
  },
  "gift_tender_total": {
    "amount": 0
  },
  "third_party_gateway_payments": {
    "amount": 0,
    "subcategories": []
  }
}
  ```


# Authentication

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# Kittens

## Get All Kittens

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

