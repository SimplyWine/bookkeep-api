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

This example API documentation page was created with [Slate](https://github.com/slatedocs/slate).

# Summary Composition

To create a daily summary for a revenue cloud app you will need to understand some concepts.

## Input tells your code what report to generate


```javascript
"inputs": {
  "summary_date": "2021-02-19",
  "bk_client_key": "jt-bottle-shop-2839@bookkeep.com",
  "bk_organization_id": "9zOe70LNPwlN",
  "location_id": "LST5S4CDVQVAC",
  "shop_domain": "my-shop-subdomain"
  "bk_external_id": 5823,
  "qbo_realm_id": "4fa9b94c-44b8-4ae5-b8b2-98ae807956a2",
  "journal_entry_template": "square_summary",
  "location": "Boardwalk",
  "start_of_day": "2021-02-19T00:00:00-08:00",
  "testing": true
 }
 ````
We send an input which will give your code instruction on what to run.  

Name |  Description
--------- | -----------
summary_date  | The report date for this report. Should be used to calcuate the `short_summary` date and also in `source_uuid` this is the date to be used. 
bk_client_key | email address of the organizations inbox.  Not used in your code can be ignored.
bk_organization_id | the unique id of the organization.  Is used in `source_uuid` 
location_id | depending on the app this will indicate a unique location in the source system for your summary.
shop_domain | depending on the app this will indicate a unique shop name and is used in `source_uuid` and to create the `source_report_url`
bk_external_id | can be ignored
qbo_realm_id | This represents the accounting connection and is passed back in `post_raw_data`
journal_entry_template | This triggers which report summary you will run and post back
location | A text field for the user to input what they want to call the report location.  Should be included in `short_summary`
start_of_day | the beginning point of the 24 hour summary.  Includes timezone at time of report run.  However you should convert to the timezone of the source system account. 
testing | if this report is for testing or live.  

<aside class="notice">
The timezone in the input is the timezone that should be used to pull the 24 hour period.  The `start_of_day` is the beginning of the 24 hour period to query. Sometimes the input timezone will be incorrect.  Should the source system have a timezone setting you should update the timezone to match the source system timezone otherwise use the input timezone given.
</aside>

## Which orders do we summarize?

We are doing accrual accounting which means that we summarize for the day the payment was made against the order.  Often this required pulling the payments first and then backing into the orders.  Or pulling orders which are completed or paid.  

<aside class="notice">
When pulling orders always round on the line item level before adding. If you are capturing line items of quantity * price you need to round for each line before adding.  Always round to 2 digits.  Never add up a column of totals without first rounding each field to 2 digits.  
</aside>

Any returns or exchanges processed on the summary date are also included since they have a finanical impact on that summary date.  

This means pulling any orders modified on the summary date to find refunds or exchanges.  Or else looking at the finanical transactions to find orders updated that day. 

Also note that if an order has not been paid for meaning it's pending or similiar status then it should be ignored in the financial summary.  

### Grouping of summaries

There are 2 ways we group summaries.

1. Currency - Journal entries required a single currency. 
2. Channel - Channel is sent as an option in the input to specify a channel or groups of channels to include in the summary. This could be a store_id or name of a channel.  

## json Body
```javascript
{
  "bk_organization_id": "inputs.bk_organization_id",
  "journal_entry_template": "squarespace_summary",
  "source_uuid": "inputs.journal_entry_template + '-' + inputs.bk_organization_id + '-' + inputs.shop_domain + '-' + inputs.report_date +  inputs.channel_ids",
  "bk_external_id": "inputs.bk_external_id",
  "short_summary": "Square Summary Boardwalk Saturday, February 6th, 2021 ${{net_sales}}",
  "post_raw_data": {...},
  "source_raw_data" {...},
  "testing":true
}
```

The body you post to the API consists of the following. 

Name | Required? | Description
--------- | ------- | -----------
bk_organization_id  | true | is sent in the input to the job or else stored for the organization. 
journal_entry_template | true | sent in the input and is the key that select which report to generate.  
source_uuid | true | The unique key for the summary and must be composed properly. 
bk_external_id | true | another id usually sent in the input.
short_summary | true | a human readable summary of what we are posting for email subject line and for reporting summary lines. It should start with the template name then `location` from `input` the day of the week spelled out completely, then the Month and date.  Then the currency symbol for the summary and finally the NET sales or deposit amount or fees amount depending on template. Be sure your date is formatted as shown with commas.  Also be very sure to use `input` currency and 2 digits after decimal. 
post_raw_data | true | the main report and will be explained below.
source_raw_data | true |  used for troubleshooting data it is a text field and can include things like the list of orderids, or calculation tests. Its for us to troubleshoot with live data so put anything in there that will help.  
testing | true | When set to `true` you will a receive more detailed response with tests against the json body sent. Also when `true` the record will not be sent to accounting. 


## Source UUID
```javascript
  { ...
    "source_uuid":"squarespace_summary-3n0PJz2bPYDE-mydomain-2021-02-01-USD-channel123-channel345"
  }
  ```


The `source_uuid` is what makes each posting unique in the database so it must be composed properly to prevent duplicates.  It is a string of fields connected by `-` 

The first part of the string is the `input.journal_entry_template`

Next is the unique `input.bk_organization_id`

Then a connection id or store domain from `inputs.domain` or `inputs.connection_id` do not inlude `https` or `:` or `\\` in this string or it will break. If the posting is a payout or deposit it should include the unique ID for the deposit or payout.  

Next comes the report date in iso format eg. `2021-02-01`

then we include the 3 character currency code like `USD`

And finally if the report is filtered to channels those channel ids should be appended to the end separated by `-` but if no channels are in the input the dash should not display. 


## post_raw_data
```javascript
 {
  "inputs": {},
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
    "amount": 100.00,
    "subcategories" :
      { "label": "Tee Shirts",
        "amount": 20.00
      },
      { "label": "Clothing",
        "amount": 80
      }
  },
   "shipping_income": {
    "amount": 10.00
  },
  "discounts": {
    "amount": 2.00,
    "subcategories" :
      { "label": "Tee Shirts",
        "amount": 2.00
      }
  },
  "returns": {
    "amount": 5.00
  },
  "gift_cards_issued": {
    "amount": 30
  },
  "gratuity_collected": {
    "amount": 15
  },
  "sales_tax_collected": {
    "amount": 8.85
  },
  "stripe_payments": {
    "amount": 100
  },
  "other_payments": {
    "amount": 50
  },
  "gift_cards_redeemed": {
    "amount": 11.85
  }
}
  ```

The `post_raw_data` is the main report for the post to the api. 

Things to note:

1. This is an example but each template may have different node names.  We try to stick to standards below.  
1. Fields required are `inputs`, `je_date`, `je_type`, `currency`, `realm_id`, `build_number`, `bk_external_id`, `source_report_url`, `je_private_note`
1. The financial numbers can have different names but mostly we standardize on `gross_sales`, `discounts`, `returns`, `gift_cards_issued`, `gift_cards_redeemed`, `sales_tax_collected`, `gratuity_collected`, `other_payments`
1. each financial node has `amount` and if subgroupings are present then `subcategories`
1. If there are no grouping breakdowns under a node then `subcategories` should not display.

## Numbers should balance

```
 +        100.00 gross_sales
 +         10.00 shipping_income
 -          2.00 discounts
 +         30.00 gift_cards_issued
 +         15.00 gratuity_collected
 +          8.85 sales_tax_collected
 --------------- 
 +        161.85 



 +        100.00 stripe_payments
 +         50.00 other_payments
 +         11.85 gift_cards_redeemed
 --------------- 
 +        161.85 
 ```
<aside class="success">
  `income + libilities = payments`
</aside>

## Swedish Rounding

In countries like canada where change doesn't go to the penny and swedish rounding is used we put the `swedish_rounding` amount into the discounts `subcatgegories` and into the discount amount. 

<aside class="notice">
You should build tests on your `post_raw_data` field to check numbers before you send.  
</aside>


Test to build:

1. Each subcategory node must equal or be less than the higher level `amount` it can never sum to more or the posting will break.
1. The finanical numbers should balance in the format `income + libilities = payments`
1. Make sure all subcategories amounts sum to equal or less than the higher level `amount`.  They can never be more.  In addition negative numbers in the subcategories should rarely happen.  Only for sales tax.  



## Sales Tax Node (sales_tax_collected)
```javascript
  { 
     "sales_tax_collected": 
      "amount": 999.99
      "subcategories":  
      { 
        "rate": 0.08875,
        "label": "US-NY-Sales Tax 8.875%-8.87500",
        "amount": 226.85,
        "taxablesales": 2556.06
      },
      {
        "rate": 0.06,
        "label": "US-CA-CA STATE TAX-6",
        "state": "CA",
        "amount": 391.12,
        "country": "US",
        "taxablesales": 6518.67
      },
      {
        "rate": 0.005,
        "label": "US-CA-CA SPECIAL TAX-0.5",
        "state": "CA",
        "amount": 12.54,
        "country": "US",
        "taxablesales": 2508.00
      },
      {
        "rate": 0.0025,
        "label": "US-CA-CA COUNTY TAX-0.25",
        "state": "CA",
        "amount": 16.27,
        "country": "US",
        "taxablesales": 6508.00
      }
  }
```

Sales tax is broken down and summarized by the label groups in the `subcategories` node as follows:

<aside class="notice">
Always round to 2 digits.
</aside>

Name |  Description
--------- | -----------
rate  | is decimal format out to 5 digits and comes from the source system
label | should follow this format `{2 char country code}-{2 char province/state code}-{Name Here}-{rate as percentage to 5 digits}` where the first 2 characters are the country code, the next 2 are a province code and the next is the name from the source system and finally the rate as a percentage.
amount | the actual amount from the source system grouped by the label above.
taxablesales | calculated as `amount  / rate` and should be rounded to 2 digits.
state | if a state or province code is available add that field
country | if a country code is available add that field



## Test Posting

When you have created your summary you can post tests to this api for inspection.
`https://gateway.bookkeep.com/api/v1/posts`

You will need to email the bookkeep team to request credentials.  

```javscript
{
  "request": {
    "url": "https://gateway.bookkeep.com/api/v1/posts",
    "method": "POST",
    "body": "...",
    "headers": {
      "Authorization": "Token mytoken, api_key=MY_API_KEY",
      "Content-Type": "application/json",
    }
  }
  ```

<aside class="warning">Everything below here is not finished yet.</aside>

# Deposit Composition

<aside class="notice">
A deposit is the same as a settlement or a payout.  We use those words interchangably. 
</aside>

A deposit is an accounting posting that needs to match to the bank account.



# Authentication


<aside class="notice">
This section is not updated yet.
</aside>

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>



### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve


