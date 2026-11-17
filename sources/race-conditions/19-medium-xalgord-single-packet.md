---
title: "Winning the Race: The Single-Packet Attack on Web Race Conditions"
author: Krishna Kumar (xalgord)
url: https://xalgord.medium.com/winning-the-race-the-single-packet-attack-on-web-race-conditions-e04e8fc1dc62
date_published: 2026-10-22
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

CAPTURE STATUS: PAYWALLED / PARTIAL. This is a Medium "Member-only story". Only the public preview (TL;DR plus the first section and the start of the second) was available. The rest of the article was NOT retrieved and no paywall bypass was attempted. 9 min read.

# Winning the Race: The Single-Packet Attack on Web Race Conditions

By Krishna Kumar (Penetration tester | Bug Bounty Hunter). Oct 22, 2026. Tags: Cybersecurity, Bug Bounty, Web Security, Application Security, Penetration Testing.

_Why "it worked once" is the only proof that matters, and how to make a race condition happen on command with a single TCP packet._

**TL;DR:** Race conditions are the bugs most scanners wave at and never confirm, because the only way to know a race is real is to win it. In this writeup I break down what a _race window_ actually is, why network jitter made remote races feel flaky for years, and how James Kettle's _single-packet attack_ collapses 20-30 HTTP/2 requests into one TCP packet to kill that jitter. Then I walk the **predict, probe, prove** methodology with real Turbo Intruder and Burp Repeater workflows, the six race-condition classes that actually pay, a CVSS 3.1 score, and atomic-fix code. The throughline: detection is a guess; a verified exploit is proof.

## What a race condition actually is

A race condition exists when an application processes concurrent requests against the same data without adequate safeguards, so two operations collide and produce behaviour the developers never intended. The dangerous slice of time is the **race window**, the gap between the moment the app _checks_ something and the moment it _acts_ on it.

The textbook example is a balance: the server reads your balance and confirms it's enough (the check), then decrements it and ships the goods (the act). If two requests both clear the check before _either_ reaches the act, both are authorised to spend the same money. This family of bugs is a **time-of-check to time-of-use (TOCTOU)** flaw: the state you validated is no longer the state you're mutating.

It sounds niche. It isn't. Single-use discount codes, gift cards, "claim your free trial once", withdrawal limits, invite quotas, vote-once polls, rate limits on OTP and login, even multi-factor steps: all of them are a check followed by an act, and all of them are racing targets.

[Figure: Timeline diagram of a race window. Request A and Request B both pass the CHECK point before either reaches the ACT point, inside a highlighted race window, so a one-time value is spent twice.]

## Why race conditions are so hard to prove

Locally, racing is trivial: two threads, shared memory, done. Over the network it's a different...

[TRUNCATED BY PAYWALL: "Create an account to read the full story. The author made this story available to Medium members only."]
