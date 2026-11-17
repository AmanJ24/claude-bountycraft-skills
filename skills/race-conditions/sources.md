# Race Conditions skill: sources

Every claim in `SKILL.md` and `references/` should trace back to an entry here. Per-writeup notes are in `../../extracted/race-conditions/`. 20 sources scraped (19 extracted; #19 was a paywalled preview, dropped as a redundant single-packet explainer). For this class the research spine (01-09) carries the technique; the HackerOne reports (10-17) supply the sub-variant diversity. All 8 H1 reports were publicly readable this run.

| # | Title | Author | URL | Date | Disclosed report | Bounty | Extracted note |
|---|-------|--------|-----|------|------------------|--------|----------------|
| 01 | Smashing the state machine: the true potential of web race conditions | James Kettle (PortSwigger) | https://portswigger.net/research/smashing-the-state-machine | 2023-08-09 | none (CVE-2022-4037) | n/a (research) | `extracted/race-conditions/01-portswigger-smashing-state-machine.md` |
| 02 | The single-packet attack: making remote race-conditions 'local' | James Kettle (PortSwigger) | https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local | 2023-10-18 | none | n/a (research) | `extracted/race-conditions/02-portswigger-single-packet-attack-local.md` |
| 03 | Race conditions (Web Security Academy topic) | PortSwigger | https://portswigger.net/web-security/race-conditions | unknown | none | n/a (reference + labs) | `extracted/race-conditions/03-portswigger-race-conditions-topic.md` |
| 04 | Beyond the Limit: Expanding single-packet race condition with a first sequence sync | RyotaK (Flatt Security) | https://flatt.tech/research/posts/beyond-the-limit-expanding-single-packet-race-condition-with-first-sequence-sync/ | 2024-08-02 | none | n/a (research) | `extracted/race-conditions/04-flatt-ryotak-first-sequence-sync.md` |
| 05 | Concurrency exploits: the ultimate Bug Bounty guide to race conditions | Alex Brumen (YesWeHack) | https://www.yeswehack.com/learn-bug-bounty/ultimate-guide-race-condition-vulnerabilities | 2025-09-26 | none | n/a (reference) | `extracted/race-conditions/05-yeswehack-race-condition-guide.md` |
| 06 | Race Conditions and Where to Find Them | Themis Zoumpoulakis (Hackcraft) | https://www.hackcraft.gr/2026/04/race-conditions-and-where-to-find-them/ | 2026-04-07 | none (refs H1 #429026, #1913309, CVE-2022-4037) | n/a (reference) | `extracted/race-conditions/06-hackcraft-race-conditions-where-to-find.md` |
| 07 | One Bug, Many Faces: Understanding Every Type of Race Condition Vulnerability | Advik Kant (deoxys) | https://dev.to/deoxys/one-bug-many-faces-understanding-every-type-of-race-condition-vulnerability-4po3 | 2025-08-02 | none | n/a (reference) | `extracted/race-conditions/07-devto-deoxys-race-types.md` |
| 08 | Race Condition Testing (Turbo Intruder docs) | PortSwigger | https://github.com/PortSwigger/turbo-intruder/blob/master/docs/race-conditions.md | unknown | none | n/a (tooling) | `extracted/race-conditions/08-github-turbo-intruder-race-docs.md` |
| 09 | Dive into Single Packet Attack | Amin Nasiri (nxenon) | https://infosecwriteups.com/dive-into-single-packet-attack-3d3849ffe1d2 | 2023-11-24 | none | n/a (reference) | `extracted/race-conditions/09-medium-amin-nasiri-single-packet.md` |
| 10 | Race Condition in Redeeming Coupons (Instacart) | see report | https://hackerone.com/reports/157996 | 2016 | same | awarded x2, amounts hidden | `extracted/race-conditions/10-h1-157996-coupon-redeem-race.md` |
| 11 | Race condition in faucet (Cosmos/Starport) | cyberboy (+ z0mb13) | https://hackerone.com/reports/1438052 | 2022 | same | $5,000 ($2,500 + $2,500); Critical 9.3 | `extracted/race-conditions/11-h1-1438052-cosmos-faucet-race.md` |
| 12 | Race Condition in account survey (Slack) | see report | https://hackerone.com/reports/165570 | 2016 | same | awarded, amount hidden | `extracted/race-conditions/12-h1-165570-slack-account-survey-race.md` |
| 13 | Race Condition when following a user (Every.org) | see report | https://hackerone.com/reports/927384 | 2020 | same | awarded, amount hidden; Low | `extracted/race-conditions/13-h1-927384-follow-user-race.md` |
| 14 | Race Condition leads to undeletable group member (HackerOne/hacker101) | see report | https://hackerone.com/reports/604534 | 2019 | same | awarded, amount hidden; Low 2.4 | `extracted/race-conditions/14-h1-604534-undeletable-group-member-race.md` |
| 15 | Race condition on action: Invite members to a team (Omise) | see report | https://hackerone.com/reports/1285538 | 2021 | same | awarded, amount hidden; Low | `extracted/race-conditions/15-h1-1285538-invite-team-race.md` |
| 16 | Race condition in performing retest (HackerOne) | see report | https://hackerone.com/reports/429026 | 2018 | same | ~$2,100 net; Medium 4.2 | `extracted/race-conditions/16-h1-429026-retest-race.md` |
| 17 | Race Condition in Flag Submission (hacker101 CTF) | see report | https://hackerone.com/reports/454949 | 2019 | same | awarded, amount hidden | `extracted/race-conditions/17-h1-454949-flag-submission-race.md` |
| 18 | Exploiting Race Conditions for Infinite Discounts | Aditya Bhatt | https://github.com/AdityaBhatt3010/Bug-Bounty-Race-Exploiting-Race-Conditions-for-Infinite-Discounts | unknown | none | n/a (practitioner lab walkthrough) | `extracted/race-conditions/18-github-adityabhatt-infinite-discounts-race.md` |
| 20 | Bug bounty handbook: race conditions (chapter) | mirshad0w | https://github.com/mirshad0w/bug-bounty-handbook/blob/main/22-race-conditions.md | unknown | none | n/a (reference) | `extracted/race-conditions/20-github-mirshad0w-race-conditions-handbook.md` |

Dropped: #19 "Winning the Race: The Single-Packet Attack" (xalgord, Medium) was members-only; saved the public preview only (not bypassed) and dropped from extraction as a redundant single-packet explainer (covered by 01, 02, 09).

Notes:
- #18's real-world examples (Tesla, Uber) are the author's uncited claims; used only as a practitioner lab walkthrough, flagged in the note.
- #04 (first-sequence-sync) is the expert escalation; its note was written by hand after a Sonnet safeguard blocked the automated extraction (legitimate published research).
- CVE cross-refs in the research: CVE-2022-4037 (GitLab email-verify forgery), CVE-2024-58248 (nopCommerce gift-card reuse).
