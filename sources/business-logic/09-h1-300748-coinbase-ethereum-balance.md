---
title: "Ethereum account balance manipulation"
author: vicompany
url: https://hackerone.com/reports/300748
date_published: 2017-12-27
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: https://hackerone.com/reports/300748
bounty: not stated (bounty awarded Jan 26, 2018, amount hidden on page)
vuln_class: business-logic
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

#300748 Ethereum account balance manipulation (Coinbase)

Status: Resolved. Severity: Critical (9 ~ 10). Weakness: Business Logic Errors. Reported Dec 27, 2017 08:38 UTC. Disclosed Mar 21, 2018.

**Summary by Coinbase**

The researchers noticed an issue with our ETH receiving code when receiving from a contract. This allowed sending of ETH to Coinbase to be credited even if the underlying contract execution failed. The issue was fixed by changing the contract handling logic. Analysis of the issue indicated only accidental loss for Coinbase, and no exploitation attempts.

The Security team thanks @vicompany for the quick disclosure, and also the internal team for pushing a fix within hours. We do appreciate @vicompany's patience as the full communication loop back to HackerOne took significantly longer than the fix deployment cycle.

**Summary by vicompany**

**Short Summary:**
By using a smart contract to distribute ether over a set of wallets you can manipulate the account balance of your Coinbase account. If 1 of the internal transactions in the smart contract fails all transactions before that will be reversed. But on Coinbase these transactions will not be reversed, meaning someone could add as much ether to their balance as they want. When you look up the Coinbase wallet address after this transaction you will see that it is empty, but checking your Coinbase wallet will show your funds.

**Steps To Reproduce:**

- Setup a smart contract with a few valid Coinbase wallets and 1 final faulty wallet (always throw exception when receiving funds smart contract for example)
- Transfer appropriate funds to smart contract.
- Execute smart contract adding the set amount of ether to the Coinbase wallets without ever actually leaving the smart contract wallet because the complete transaction fails at the last wallet.
- Repeat until you have more than enough ethereum in your Coinbase wallet.
- Cash out, transfer to off site wallet

For some more information see
https://www.vicompany.nl/magazine/from-christmas-present-in-the-blockchain-to-massive-bug-bounty

## Timeline (as shown on page)

- Dec 27, 2017 08:38 UTC: vicompany submitted report to Coinbase.
- Dec 27, 2017: several comments by vicompany and cbsisl (Coinbase). Comment bodies not shown in public view.
- Jan 11, 2018 19:08 UTC: cbsisl changed status to Triaged; 19:09 Needs more info; 19:21 vicompany changed to New.
- Jan 26, 2018 01:11 UTC: Coinbase rewarded vicompany with a bounty (amount hidden).
- Jan 26, 2018 01:11 UTC: cbsisl closed report as Resolved.
- Feb 19, 2018: vicompany requested disclosure.
- Mar 21, 2018: report disclosed.
- Apr 3-4, 2018: final comments by cbsisl and vicompany.

Participants: vicompany, cbsisl. Bounty, retest, total reward: Hidden. CVE: None.

Capture note: public page shows summaries and timeline only; individual comment bodies were not rendered. Treat as full public content available (no paywall).
