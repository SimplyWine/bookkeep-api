---
title: bookkeep.com API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell

toc_footers:

includes:
  - errors

search: true

code_clipboard: true
---

# Introduction

Welcome to the documentation for bookkeep.com's accounting integration platform.

This example API documentation page was created with [Slate](https://github.com/slatedocs/slate).

Note that this is a work in progress and only available to early partners.  

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

<aside class="notice">
What if no orders, refunds or exchanges are found durning the time period?  In this situation we want the user to know our system ran propelry for the time period so post an entry to the gateway with `post_raw_data` with a 0 for each top line node `amount`
</aside>

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
  "testing":true,
  "transactions_summarized_count":999,
  "summarized_net_sales":99.99,
  "apify_run_url":"https://job-system.com/job_run_url",
  "attachments": [{
    "body": "\"AmazonOrderId\",\"SellerOrderId\",\"MarketplaceName\",\"PostedDate\",\"SellerSKU\",\"OrderItemId\",\"QuantityShipped\",\"CurrencyAmount\",\"CurrencyCode\",\"Type\",\"TaxCollectionModel\"\n\"https://sellercentral.amazon.com/orders-v3/order/112-5952175-4476226\",\"https://sellercentral.amazon.com/orders-v3/order/112-5952175-4476226\",\"Amazon.com\",\"2021-05-05T13:51:32Z\",\"PG-SCNT-4-FBA-2\",\"10563975951450\",\"1\",\"22.0\",\"USD\",\"Principal\",\"\"\n\"",
    "filename": "Wednesday, May 5th, 2021.csv",
    "content_type": "text/csv"
  }]
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
attachments | false | an array of object which are attachment data [body,content_type,file_name]. body in plaintext
transactions_summarized_count| false | count of orders and refunds summarized add refund count to orders.
summarized_net_sales| false| the net sales (gross sales - discounts - refunds) for this report. 

<aside class="notice">
deposit not found should still create an entry in gateway with post_raw_data. The `source_uuid` will not have a deposit id which is ok.  The `post_raw_data` should show amount=0 and the  `short_summary` will say "no deposit today"  at the end.
</aside>


## Source UUID
```javascript
  { ...
    "source_uuid":"squarespace_summary-3n0PJz2bPYDE-mydomain-2021-02-01-USD-channel123-channel345"
  }
  ```


The `source_uuid` is what makes each posting unique in the database so it must be composed properly to prevent duplicates.  It is a string of fields connected by `-` 

The first part of the string is the `input.journal_entry_template`

Next is the unique `input.bk_organization_id`

Then a connection id or store domain from `inputs.domain` or `inputs.connection_id` do not inlude `https` or `:` or `\\` in this string or it will break. 

In the case of a payout or deposit the unique payout or deposit ID is used. If no deposits found on the date in question then use `no-deposit` in that space.  Note that a summary will not have any unique ids since we generate that summary in our code (there will only be one per day since its a summary) which is why we skip this space when composing a source_uuid for the summaries. 

Next comes the report date in iso format eg. `2021-02-01`

then we include the 3 character currency code in ISO format `USD`

Options: AED, AFN, ALL, AMD, ANG, AOA, ARS, AUD, AWG, AZN, BAM, BBD, BDT, BGN, BHD, BIF, BMD, BND, BOB, BRL, BSD, BTC, BTN, BWP, BYR, BZD, CAD, CDF, CHF, CLP, CNY, COP, CRC, CUC, CUP, CVE, CZK, DJF, DKK, DOP, DZD, EGP, ERN, ETB, EUR, FJD, FKP, GBP, GEL, GGP, GHS, GIP, GMD, GNF, GTQ, GYD, HKD, HNL, HRK, HTG, HUF, IDR, ILS, IMP, INR, IQD, IRR, IRT, ISK, JEP, JMD, JOD, JPY, KES, KGS, KHR, KMF, KPW, KRW, KWD, KYD, KZT, LAK, LBP, LKR, LRD, LSL, LYD, MAD, MDL, MGA, MKD, MMK, MNT, MOP, MRO, MUR, MVR, MWK, MXN, MYR, MZN, NAD, NGN, NIO, NOK, NPR, NZD, OMR, PAB, PEN, PGK, PHP, PKR, PLN, PRB, PYG, QAR, RON, RSD, RUB, RWF, SAR, SBD, SCR, SDG, SEK, SGD, SHP, SLL, SOS, SRD, SSP, STD, SYP, SZL, THB, TJS, TMT, TND, TOP, TRY, TTD, TWD, TZS, UAH, UGX, USD, UYU, UZS, VEF, VND, VUV, WST, XAF, XCD, XOF, XPF, YER, ZAR and ZMW. Default is USD.

And finally if the report is filtered to channels those channel ids should be appended to the end separated by `-` but if no channels are in the input the dash should not display. 


## post_raw_data
```javascript
 {
  "inputs": {},
  "je_date": "2021-01-29",
  "je_type": "my_summary",
  "currency": "USD",
  "realm_id": "9130348211837726",
  "docnumber": "OptionalIfMultiplePostsOnSameDay",
  "api_version": "2020-02-04",
  "build_number": "0.1.383",
  "bk_external_id": 4584,
  "job_run_url":"https://this_is_the_job_run.com/runid",
  "source_report_url": "my-source-system.com/comapre-numbers-to-this-report.html",
  "source_location_name": "Name of this location from source system",
  "source_location_id": "Source system id for this location",
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
  "gross_sales_note":"24 Orders 1 Refund from 2021-01-29 00:00:00 to 2021-01-30 00:00:00"
  ,
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

Standardized fields:

Name | Required? | Description
--------- | ------- | -----------
{node}_note | false | A text note that will display on the line in the Journal Entry in QuickBooks Online or Xero. 
source_location_name  | false | The source system name for the location or channel or combination of both if available 
source_location_id | false | A location id in the source system for the name if available
source_report_url | true | The report in the source system that corresponds to the data in the job summary. 
je_private_note | true | could be '' but this is the note that passes through to the entry in accounting.
docnumber | false | Optional field in case there are mutiple summaries per day this will make unique in QBO docnumber field
inputs | true | the inputs to run your job excluding any access tokens.
je_date | true | the report date from the input.  
je_type | true | The template name from the input.
currency | true | currency of the data summary
realm_id | true | this is found in the input
build_number | true | this is the code build that is running the job. 
bk_external_id | true | from inputs
job_run_url | false| the apify job url or other job url for this post_raw_data so we can refer back to it
transactions_summarized_count | true | total orders or transactions summarized for this summary and includes refunds
summarized_net_sales | true | total net sales summarized if this is a sales summary if not then 0

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

## Extra Helpful Info
<aside class="notice"
Posting summary information can be disorienting when it appears as a journal entry with no context. So to help with this situation, we need to add context.  The places to do this are in the `je_private_note` the `source_report_url` and in any node filed with `_note` attached.  In addition, adding an attachment of a csv file with all transactions summarized is extremely helpful. 

Examples of extra helpful context are:
The url of a report specifically for the date period of the summary and the currency and time zone so that it matches exactly to the summary.  That url should go into `source_report_url` and also into `je_private_note`. 
Adding a `_note` to any node will add that text next to the exact number.  A great place to use this is for deposits and expected deposits.  An example of a helpful entry is below. 

`Gross Amount $418.34 from 12 transactions from 2021-06-13 05:08 to 2021-06-13 21:09`
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

```avscript
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

# Deposit Composition

```javscript
{
  "je_type": "shopify_payments_deposit",
  "api_version": "2020-02-04",
  "bk_external_id": "6329",
  "je_date": "2021-02-18",
  "realm_id": "9130349917335986",
  "balance_reduction": {
    "amount": 662.75
  },
  "adjustment": {
    "amount": 0
  },
  "bank_account_deposit": {
    "amount": 643.23
  },
  "fees_reduction": {
    "amount": 19.52
  },
  "loan_payment_reduction": {
    "amount": 0
  },
  "other_deposit_withheld": {
    "amount": 0
  },
  "currency": "USD",
  "je_private_note": "https://maine-fly-company.myshopify.com/admin/payments/payouts/64298877108\n ",
  "source_report_url": "https://maine-fly-company.myshopify.com/admin/payments/payouts/64298877108",
  "inputs": {
    "summary_date": "2021-02-20",
    "bk_client_key": "test-3358@bookkeep.com",
    "bk_organization_id": "K3bPA0MKeQA2",
    "shop_domain": "maine-fly-company.myshopify.com",
    "bk_external_id": 6329,
    "qbo_realm_id": "9130349917335986",
    "journal_entry_template": "shopify_payments_deposit",
    "location": "",
    "start_of_day": "2021-02-18T00:00:00-05:00",
    "testing": true
  },
  "build_number": "unable_to_fetch"
}
```

<aside class="notice">
A deposit is the same as a settlement or a payout.  We use those words interchangably. 
</aside>

A deposit is an accounting posting that needs to match to the bank account. The `bank_deposit` field in `post_raw_data` is the amount that will hit the bank account. There can be other fees lines and of course also the gross amount which we calculate for the `balance_reduction`.  

Name | Required? | Description
--------- | ------- | -----------
balance_reduction  | true | The amount the source balance is reduced in total. 
bank_account_deposit | true | The amount deposited to the bank account.
fees_reduction | true | Amount of fees removed for the specific deposit or potentially all the orders associated with the deposit. 
loan_payment_reduction | false | A loan payment that was removed from the Deposit and balance.
other_deposit_withheld | false | Optional field in case there are other items removed from deposits.  Subcategories should be used.
inputs | true | the inputs to run your job excluding any access tokens.
je_date | true | the report date from the input.  
je_type | true | The template name from the input.
currency | true | currency of the data summary
realm_id | true | this is found in the input
build_number | true | this is the code build that is running the job. 
bk_external_id | true | from inputs
currency | true | currency of the deposit.
source_location_name  | false | The source system name for the location or channel or combination of both if available 
source_location_id | false | A location id in the source system for the name if available
source_report_url | true | The report in the source system that corresponds to the data in the job summary. 
je_private_note | true | could be '' but this is the note that passes through to the entry in accounting.
docnumber | false | Optional field in case there are mutiple deposits per day this will make unique in QBO docnumber field

# Setup Script Composition

The setup call helps get subcategories for mapping the templates.  Subcategories would be for gross_sales, sales_tax_collected, and other_tender nodes.

# Authentication




