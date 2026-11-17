# Race Conditions: candidate writeups (Find + Filter steps)

Find/Filter done 2026-11-17. For this class the TECHNIQUE is as central as the findings: the
single-packet attack (Kettle, 2023) made remote races reliably exploitable, so the research spine is
weighted heavily. Covet sub-variant diversity: limit overrun (redeem/withdraw/vote past a limit),
multi-endpoint collision, single-endpoint collision (same request in parallel), partial-construction /
time-sensitive sub-states, rate-limit / OTP-brute bypass via race, payment/balance double-spend,
social-action races (follow/invite/membership), auth-flow / email-confirmation token races.
Target: 15 to 25 kept. Ask before scraping more than ~25 pages in one run. (20 selected.)

Status: candidate | keep | drop | scraped

| #  | Title | Author | URL | Platform | Sub-variant | Status |
|----|-------|--------|-----|----------|-------------|--------|
| 01 | Smashing the state machine: the true potential of web race conditions | James Kettle (PortSwigger) | https://portswigger.net/research/smashing-the-state-machine | portswigger | THE paper: single-packet attack, sub-states, multi/single-endpoint, predict-probe-prove | candidate |
| 02 | The single-packet attack: making remote race-conditions local | James Kettle (PortSwigger) | https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local | portswigger | technique: protocol applicability (H1/H3/WebSocket/SMTP) | candidate |
| 03 | Race conditions (Web Security Academy topic) | PortSwigger | https://portswigger.net/web-security/race-conditions | portswigger | taxonomy + methodology + lab list | candidate |
| 04 | Beyond the Limit: Expanding single-packet race condition with first sequence sync | RyotaK (Flatt Security) | https://flatt.tech/research/posts/beyond-the-limit-expanding-single-packet-race-condition-with-first-sequence-sync/ | personal-blog | EXPERT technique: 10k requests, OTP/token-brute via race, IP frag + seq reorder | candidate |
| 05 | Ultimate Bug Bounty guide to race condition vulnerabilities | YesWeHack | https://www.yeswehack.com/learn-bug-bounty/ultimate-guide-race-condition-vulnerabilities | other | methodology + Turbo Intruder template | candidate |
| 06 | Race Conditions and Where to Find Them | Themis Zoumpoulakis (Hackcraft) | https://www.hackcraft.gr/2026/04/race-conditions-and-where-to-find-them/ | personal-blog | practical walkthrough: limit-overrun, multi-endpoint, benchmarking, connection warming | candidate |
| 07 | One Bug Many Faces: Understanding Every Type of Race Condition Vulnerability | deoxys (dev.to) | https://dev.to/deoxys/one-bug-many-faces-understanding-every-type-of-race-condition-vulnerability-4po3 | other | taxonomy of race types | candidate |
| 08 | Turbo Intruder race-conditions.md (tooling docs) | PortSwigger | https://github.com/PortSwigger/turbo-intruder/blob/master/docs/race-conditions.md | github | tooling reference (Engine.BURP2, gate, single-packet) | candidate |
| 09 | Dive into single-packet attack | Amin Nasiri | https://infosecwriteups.com/dive-into-single-packet-attack-3d3849ffe1d2 | medium | practitioner explainer of the single-packet attack | candidate |
| 10 | Race Condition in Redeeming Coupons | — | https://hackerone.com/reports/157996 | hackerone | limit overrun: coupon redeemed multiple times | candidate |
| 11 | Race condition in faucet (Cosmos) | z0mb13 | https://hackerone.com/reports/1438052 | hackerone | balance/faucet over-claim ($2,500) | candidate |
| 12 | Race Condition in account survey (Slack) | — | https://hackerone.com/reports/165570 | hackerone | limit/duplicate state | candidate |
| 13 | Race Condition when following a user | — | https://hackerone.com/reports/927384 | hackerone | social-action over-count (async follows) | candidate |
| 14 | Race Condition leads to undeletable group member | — | https://hackerone.com/reports/604534 | hackerone | state corruption via collision | candidate |
| 15 | Race condition on action: Invite members to a team | — | https://hackerone.com/reports/1285538 | hackerone | limit/seat overrun via invite | candidate |
| 16 | Race condition in performing retest | — | https://hackerone.com/reports/429026 | hackerone | workflow/state duplicate | candidate |
| 17 | Race Condition in Flag Submission | — | https://hackerone.com/reports/454949 | hackerone | points/limit overrun | candidate |
| 18 | Bug Bounty: Exploiting Race Conditions for Infinite Discounts | Aditya Bhatt | https://github.com/AdityaBhatt3010/Bug-Bounty-Race-Exploiting-Race-Conditions-for-Infinite-Discounts | github | discount limit overrun (practitioner) | candidate |
| 19 | Winning the Race: The Single-Packet Attack on Web Race Conditions | xalgord | https://xalgord.medium.com/winning-the-race-the-single-packet-attack-on-web-race-conditions-e04e8fc1dc62 | medium | race-window + single-packet explainer | candidate |
| 20 | Bug bounty handbook: race conditions (checklist + common mistakes) | mirshad0w | https://github.com/mirshad0w/bug-bounty-handbook/blob/main/22-race-conditions.md | github | checklist / common mistakes reference | candidate |

Notes:
- Expect some HackerOne reports (10-17) to be behind the sign-in / limited-disclosure wall (as in the
  business-logic run); the public research spine (01-09) is reliably accessible and carries the technique.
- EVAL CAVEAT: PortSwigger race labs genuinely need the single-packet attack (Burp "send group in
  parallel" / Turbo Intruder). In-page parallel fetch (Promise.all) only approximates it and will not
  reliably hit sub-millisecond race windows, so the eval phase will be partly automation-limited; the
  wide-window limit-overrun lab may still be winnable with concurrent fetch.
- Backlog if gaps surface: Keybase/Shopify/GitLab disclosed race reports; Devise auth race (from the paper).
