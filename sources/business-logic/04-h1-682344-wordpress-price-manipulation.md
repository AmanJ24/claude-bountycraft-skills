---
title: "Parameter tampering : Price Manipulation of Products"
author: ashketchum (Ashish Dhone)
url: https://hackerone.com/reports/682344
date_published: 2019-08-26
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/682344
bounty: not stated (none; closed as Duplicate of #271585)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #682344. Reported to WordPress. Status: Duplicate (of #271585). Severity: No rating. Weakness: Business Logic Errors. Disclosed Aug 29, 2019. Bounty: Hidden.

## Timeline

**ashketchum** submitted a report to WordPress. August 26, 2019, 7:17pm UTC

Hello Security Team,
I have found that you can buy any products in less amount or even we can say as free by changing the price of the product!!

POC :
1) go to https://mercantile.wordpress.org/
2) choose any product and add to cart
3) Now go to cart add your billing details
4) Intercept request with burpsuite and click on Proceed to Paypal
5) You will get the following request

REQUEST (shown with personal details trimmed):

```
GET /cgi-bin/webscr?cmd=_cart&business=payments%40hellomerch.com&no_note=1&currency_code=USD&charset=utf-8&rm=2&upload=1&return=https%3A%2F%2Fmercantile.wordpress.org%2Fcheckout%2Forder-received%2F10446%2F%3Fkey%3D[REDACTED]%26utm_nooverride%3D1&cancel_return=...&paymentaction=sale&bn=WooThemes_Cart&invoice=WP-10446&custom=%7B%22order_id%22%3A10446%2C%22order_key%22%3A%22[REDACTED]%22%7D&notify_url=https%3A%2F%2Fmercantile.wordpress.org%2Fwc-api%2FWC_Gateway_Paypal%2F&first_name=[name]&last_name=[name]&address1=[address]&city=PUNE&state=Maharashtra&zip=411057&country=IN&email=[email]&no_shipping=1&tax_cart=0.00&shipping_1=<<VULNERABLE PARAMETER>>&item_name_1=WordCamp+US+2018+T-Shirt+-+Women%27s+SM&quantity_1=1&amount_1=<<VULNERABLE PARAMETER>>&item_number_1=WP-WORDCAMPUS2018NAVY-WSM-05-106-B HTTP/1.1
Host: www.paypal.com
Referer: https://mercantile.wordpress.org/checkout/
```

Now here in this request we have two VULNERABLE PARAMETER as :
1) shipping_1
2) amount_1

Now you change amount to this parameter and forward the request .. you will be able to order the product for less price or even Free using Paypal !!

I have attached POC Video, Please take a look!!

Thank You!!
Ashish Dhone

## Impact

The attacker can reduce the price of the order and make a financial loss!!

Attachment: POC.mkv (F567164) [signed URL REDACTED]

---

**iandunn** closed the report and changed the status to **Duplicate** (#271585). August 26, 2019, 10:13pm UTC

Hi Ashish, this is a known behavior of WooCommerce, but it doesn't actually allow you to change the price; it only appears that way. IPN validation occurs after the order is placed, and will mark the order as invalid.

**ashketchum** requested to disclose this report. Aug 27, 2019.
**iandunn** agreed to disclose this report. Aug 29, 2019.
This report has been disclosed. Aug 29, 2019, 3:39pm UTC
