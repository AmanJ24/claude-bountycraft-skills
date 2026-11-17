---
title: "Race condition in faucet when using starport"
author: cyberboy (collaborator: z0mb13)
url: https://hackerone.com/reports/1438052
date_published: 2021-12-30
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/1438052
bounty: "$5,000 total shown in sidebar ($2,500 to cyberboy and $2,500 to collaborator z0mb13 in timeline)"
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Scrape status: FULL (complete report body and timeline rendered). Presigned attachment URLs with tokens replaced by [REDACTED]. Code blocks reconstructed from rendered line-numbered output.

Report #1438052: Race condition in faucet when using starport
Reported to: Cosmos. Reported by: cyberboy. Reported on: December 30, 2021, 8:45am UTC. Disclosed: July 26, 2022, 5:47pm UTC.
Status: Resolved. Severity: Critical (9.3). Weakness: None. CVE: None. Bounty: $5,000.

## Summary by cyberboy

The proper writeup of the bug can be found here at our blogpost https://blog.credshields.com/race-condition-in-tendermints-starport-7cebe176d935

The root cause of the bug was in function "Transfer" at
https://github.com/tendermint/starport/blob/7812125/starport/pkg/cosmosfaucet/transfer.go#L50-L74
We can notice in the code that each request to the faucet causes two actions to be made; one for querying the account's balance and the other for sending tokens. When sending concurrent requests to the faucet, in the time of querying the balance, for some requests, this check happens at the same time and ends up seeing less balance in the account because sending the tokens action has not been finalized for the previous requests.

## Report body (cyberboy, December 30, 2021, 8:45am UTC)

Hi team,
I and Aditya sent this bug over email on Wed, 29 Dec, 17:45 IST. Later we noticed that security reports are accepted via the HackerOne program. So, I am sending a copy of the bug report here.

## Summary:

We were testing an application and we found a race condition bug in the faucet Implementation of Starport.
https://github.com/tendermint/starport

## Steps To Reproduce:

1. Start a starport with the below configuration. Note the "coins_max" has been set to 11 tokens and hence a user cannot fetch more after the 11 token limits.

```yaml
accounts:
  - name: alice
    coins: ["0token", "200000000stake"]
  - name: bob
    coins: ["500token", "100000000stake"]
validator:
  name: alice
  staked: "100000000stake"
client:
  openapi:
    path: "docs/static/openapi.yml"
  vuex:
    path: "vue/src/store"
faucet:
  name: bob
  coins: ["5token", "100000stake"]
  coins_max: ["11token", "100000stake"]
```

2. Now call the request manually with 5 tokens per request as in our configuration after 2 requests and 10 tokens in total Alice won't be able to fetch more tokens from the faucet

```
POST / HTTP/1.1
Host: 172.105.41.242:4500
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:95.0) Gecko/20100101 Firefox/95.0
Accept: application/json
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://172.105.41.242:4500/
Content-Type: application/json
Origin: http://172.105.41.242:4500
Content-Length: 63
Connection: close
{
  "address": "ALICE_ADDRESS"}
```

Now we can confirm Alice cannot have more than 11 tokens.

3. Now regenerate the server and instead of sending a single request send a concurrent request to fetch tokens in Alice address. We used 50 requests concurrently.

[Image F1563051: Screenshot_2021-12-29_at_5.41.56_PM.png, presigned URL REDACTED]

4. Now when we check Alice balance it is 30 which should have not been more than 11

[Image F1563052: Screenshot_2021-12-29_at_5.43.09_PM.png, presigned URL REDACTED]

We believe the root cause of the issues is the go mapping which is not advised for concurrency
https://github.com/tendermint/starport/blob/develop/starport/pkg/cosmosfaucet/transfer.go#L59

## Supporting Material/References:

https://cwe.mitre.org/data/definitions/362.html

## Impact

A malicious user can send concurrent requests to fetch more tokes from faucets than the "max-credit limit" which allows.

## Timeline

- cyberboy invited another hacker as a collaborator. (December 30, 2021, 8:47am UTC)
- z0mb13 joined this report as a collaborator. (December 31, 2021, 9:34am UTC)
- cyberboy requested mediation from HackerOne Support. (February 17, 2022, 12:35pm UTC)
- **jonws** (July 21, 2022, 6:50pm UTC): Hi @cyberboy, I apologize for the delay, I am following up with the engineering team just to full cover this off and resolve and payout the bounty. Thank you for your patience and I hope to have an update for you ASAP
- jonws changed the status to **Triaged**. (July 25, 2022, 3:37pm UTC): Changing state pending payout confirmation. Issue was legitimate reading the PR and implementation, will be resolving report once confirmation is reached. Thanks for the patience!
- **cyberboy** (July 26, 2022, 7:20am UTC): Thank you, @jonws, for help here. Much appreciated!
- Cosmos rewarded z0mb13 with a **$2,500** bounty. (July 26, 2022, 3:49pm UTC)
- Cosmos rewarded cyberboy with a **$2,500** bounty. (July 26, 2022, 3:49pm UTC): Hi @cyberboy, I am happy to finally validate and reward for this, thank you so much for the detailed submission and the patience on this. I look forward to working with you in the future!
- jonws closed the report and changed the status to **Resolved**. (July 26, 2022, 3:49pm UTC)
- cyberboy requested to disclose this report. (July 26, 2022, 3:59pm UTC)
- **cyberboy** (July 26, 2022, 4:01pm UTC): Hey @jonws Thank you for the reward! As the bug was fixed, and if it is okay with your team, can we publicly disclose the bug?
- jonws agreed to disclose this report. (July 26, 2022, 5:47pm UTC)
- This report has been disclosed. (July 26, 2022, 5:47pm UTC)

Participants: cyberboy, jonws.
