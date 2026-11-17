---
title: "Race Conditions and Where to Find Them"
author: Themis Zoumpoulakis (Hackcraft)
url: https://www.hackcraft.gr/2026/04/race-conditions-and-where-to-find-them/
date_published: 2026-04-07
date_scraped: 2026-11-17
platform: other
disclosed_report: none
bounty: not applicable (research/reference)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Research and analysis by Themis Zoumpoulakis, Hackcraft’s Senior Cybersecurity Tester.

What if the most dangerous bug in your application isn't a bug at all? Every function works as expected, every test passes correctly, access controls work as intended, and parameterized database queries return the right results. Yet, two requests arriving microseconds apart cause the system to bypass a paywall, redeem a discount twice, or confirm an account registration without ever receiving the verification token. Welcome to race conditions, where "correct" code could potentially produce broken outcomes.

This blog post is based on the research presented by James Kettle in **[Smashing the State Machine: The True Potential of Web Race Conditions](https://portswigger.net/research/smashing-the-state-machine)**, which introduced the single-packet attack technique and the methodology we reference throughout. Our goal is to break down the core concepts, walk through practical exploitation using PortSwigger's labs, and cover some areas we felt were worth expanding on, including Turbo Intruder engine selection and configuration nuances, practical guidance on where to look for race conditions during security assessments and observations from our own testing experience.

## Why Race Conditions Fly Under the Radar

Race conditions are frequently overlooked in security assessments. Automated scanners cannot detect them: there is no signature to match, no payload to inject or fuzz. Even experienced testers who rely more on manual approaches tend to skip this vulnerability class, often assuming that exploitation requires a coupon mechanism or similar functionality to be present. If it's not, race conditions never make it onto the testing checklist. Identifying one requires understanding how the application manages to state across concurrent requests, which mandates manual analysis of its business logic and a willingness to consider how any multi-step operation, not just obvious validation checks, can be subverted through timing.

As modern frameworks increasingly abstract away classic attack vectors through safe defaults and parameterized functions, while static analysis and automated testing continue to raise the barrier to traditional injection-based bugs, fewer of these issues survive to production. Race conditions, however, operate at a different level entirely. They live in the gaps between functions that, individually, behave exactly as designed, but collectively produce unintended outcomes when their timing intersects.

## Commonly Affected Functionality

The most common pattern is the check-then-act sequence, commonly referred to as a TOCTOU (Time of Check to Time of Use) flaw, in which the application verifies a condition and then executes an operation based on it. But race conditions also arise in multi-step workflows and object-creation processes, where concurrent requests can interfere with intermediate application states.

**Coupon and discount code redemption** is the classic case. By landing multiple requests within the window between the validation check and the database update, the same single-use code may be redeemed more than once.

**Resource creation behind paywalls.** Applications that enforce account-tier limits, for example, restricting free-tier users to a maximum number of objects such as dashboards, projects, or workspaces may validate the current count before allowing creation. A race condition in this check allows exceeding the enforced limit without upgrading, effectively bypassing the paywall restriction entirely.

**Payment and multi-step workflows.** In applications where payment validation and order fulfillment are handled across separate steps or endpoints, a race window may exist between the moment payment is confirmed and the moment the order is finalized. An attacker can modify the cart contents, adding higher-value items or increasing quantities, after the payment check has passed but before the order is actually completed. The application processes the modified order under the assumption that the original payment validation still applies.

**Partial object construction.** Some applications create objects in multiple database operations, for example, inserting a user record and then setting its API key or confirmation token in a separate query. This introduces a brief window where the object exists but its security-critical fields are uninitialized. An attacker who races a request against this window, supplying an empty or null-equivalent value that matches the uninitialized state, can bypass verification steps entirely, such as confirming an account registration without access to the legitimate token.

## Real-World Race Condition Vulnerabilities Exploited in the Wild

- **Multiple payouts on a bug bounty platform.** A race condition in the retest feature allowed a researcher to trigger multiple payouts for the same retest by submitting concurrent retest confirmation requests. **[HackerOne \#429026](https://hackerone.com/reports/429026)**
- **Exceeding monitoring limits in Firefox Monitor.** By racing multiple requests to add email addresses, a user could bypass the per-account limit and register more addresses than allowed. [**HackerOne \#1913309**](https://hackerone.com/reports/1913309)
- **CVE-2022-4037 - Email forgery in GitLab.** A race condition in GitLab's email verification flow allowed an attacker to forge a verified email address and take over third-party accounts on services using GitLab as an OAuth provider. **[NVD](https://nvd.nist.gov/vuln/detail/CVE-2022-4037)**

## 

## Overcoming the Timing Barrier

Exploiting a race condition is fundamentally a timing problem, and the question at its core is: *"How can we actually land multiple requests at the exact same moment on the server-side?"*

The main obstacle we have to overcome is network jitter, since even a few microseconds of deviation can push a request outside the vulnerability window. Simply sending a burst of concurrent requests, whether through Burp Suite's Intruder, a threaded Python script, or tools like ffuf, and hoping two of them overlap is unreliable, since successful exploitation requires deliberately minimizing this variance.

How we do that depends on the HTTP protocol version supported by our target backend server. HTTP/1.1 and HTTP/2 handle connections and request delivery in fundamentally different ways, and each enables a distinct synchronization technique.\
In HTTP/1.1, last-byte synchronization abuses the way TCP processes partial data to align multiple requests across separate connections, reducing jitter to microseconds. In HTTP/2, the single-packet attack goes further, leveraging the protocol's multiplexing capability to pack multiple requests into a single TCP segment, eliminating network-induced jitter entirely.

It's worth noting that neither technique exploits a flaw in the HTTP protocol itself. Both leverage intended protocol behavior, specifically how each version handles connection and request delivery, to achieve the timing precision needed to trigger an application-layer race condition. Understanding both is essential, since the one we use depends on what our target supports.

## Last-Byte Synchronization (HTTP/1.1)

HTTP/1.1 enforces strict request ordering. On a single connection, each request must be fully processed and responded to before the next one begins. This head-of-line blocking means that even if we packed multiple requests into one TCP segment, the server would handle them sequentially, making a single-connection approach useless for racing. Instead, we need a separate connection per request and a way to synchronize them. This is where last-byte synchronization comes in.

The technique works as follows:

1.  N separate TCP connections are opened to the target server, one per request to be raced.
2.  Each request is sent minus its final byte. If the POST body is 200 bytes, 199 are sent. The server reads partial data and awaits the rest.
3.  The server has buffered nearly complete requests on all N connections, waiting for that last byte.
4.  All final bytes are flushed at once across every connection, as close together as possible.

## <img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/LastByteSync.drawio.png" class="alignnone wp-image-4790 size-full" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/LastByteSync.drawio.png 778w, https://www.hackcraft.gr/wp-content/uploads/2026/04/LastByteSync.drawio-300x91.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/LastByteSync.drawio-768x234.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/LastByteSync.drawio-705x215.png 705w" sizes="(max-width: 778px) 100vw, 778px" width="778" height="237" alt="race conditions" />

## Single-Packet Attack (HTTP/2)

HTTP/2 changes the picture entirely. Instead of one request per connection, HTTP/2 multiplexes numerous requests as independent streams over a single connection, with no ordering dependency between them. The server processes them concurrently.

Single-packet attack works as follows:

1.  One HTTP/2 connection is established to the target server.
2.  All competing requests are buffered locally as HTTP/2 frames but are not flushed yet.
3.  Buffered requests are written in a single call, so they get packed into one TCP segment.
4.  The server reads all frames from a single buffer and demultiplexes them into concurrent requests.
5.  Jitter is completely eliminated. The only remaining variance is how quickly the server dispatches each request to worker threads.

## <img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/SinglePacketAttack.drawio.png" class="alignnone wp-image-4792 size-full" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/SinglePacketAttack.drawio.png 830w, https://www.hackcraft.gr/wp-content/uploads/2026/04/SinglePacketAttack.drawio-300x87.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/SinglePacketAttack.drawio-768x222.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/SinglePacketAttack.drawio-705x204.png 705w" sizes="(max-width: 830px) 100vw, 830px" width="830" height="240" alt="race conditions" />

## Exploiting Race Conditions with Turbo Intruder

[**Turbo Intruder**](https://github.com/PortSwigger/turbo-intruder) is a **free** Burp Suite extension built for high-volume, timing-sensitive request delivery. It supports both last-byte synchronization and single-packet attacks out of the box, and its Jython scripting engine makes it straightforward to adapt attack templates to whatever workflow we need for our testing scenarios.

Turbo Intruder supports multiple engines; each designed for different use cases. Selecting the right engine is critical, and the choice comes down to which HTTP protocol our target server supports.

To determine which protocol is supported by our target server, we run the following command directly against the target, not through Burp's proxy, since the proxy may downgrade the connection to HTTP/1.1:

`curl -s --http2 -o /dev/null -w '%{http_version}'` [**https://target.com**](https://target.com)

A return value of 2 confirms HTTP/2 support, meaning the single-packet attack is available. A return value of 1.1 indicates the server only supports HTTP/1.1, and last-byte synchronization is the fallback.

For most cases, Engine. BURP2 is the recommended choice. It connects directly to the target, negotiates the protocol automatically via ALPN, and uses the single-packet attack over HTTP/2 when supported. If the server does not support HTTP/2, it falls back to last-byte synchronization over HTTP/1.1. If no engine is specified, the default is Engine.BURP2.

That said, Engine.BURP2 being protocol-agnostic doesn't mean configuration is irrelevant. The connection parameters must match the technique used to achieve the tightest synchronization. The examples below cover both cases.

** Security Testing Tip**\
*It's worth noting that recent versions of Burp Suite support this natively in Repeater. Grouping requests and selecting "Send group in parallel" uses the same mechanics under the hood, it relies on the BURP2 engine, negotiating the protocol and applying either the single-packet attack or last-byte synchronization depending on what the target supports. For straightforward cases, this is sufficient, but Turbo Intruder remains the more flexible option when you need fine-grained control over the engine and request parameters.*

## Exploiting a Limit-Overrun

Limit overrun race conditions are the most common and straightforward to demonstrate. To illustrate the full workflow from identification to exploitation, we'll walk through PortSwigger's **[limit overrun lab](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun)**.

In this lab, the target product costs \$1,337, but our account only has \$50 in store credit. A single application of the 20% discount code brings the price to \$1,069.60, still well beyond our credit. The goal is to exploit a race condition in the coupon redemption logic to apply the same single-use discount code multiple times, stacking the discount until the price drops below our available credit.

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/original-discount.png" class="alignnone wp-image-4795 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/original-discount.png 1194w, https://www.hackcraft.gr/wp-content/uploads/2026/04/original-discount-300x192.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/original-discount-1030x658.png 1030w, https://www.hackcraft.gr/wp-content/uploads/2026/04/original-discount-768x491.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/original-discount-705x451.png 705w" sizes="auto, (max-width: 1194px) 100vw, 1194px" width="1194" height="763" alt="race conditions" />

First, we confirm whether the target supports HTTP/2:

`curl -s --http2 -o /dev/null -w '%{http_version}'` [**https://0a15003a030d63d181ef5dbf008a0074.web-security-academy.net/**](https://0a15003a030d63d181ef5dbf008a0074.web-security-academy.net/)

The output confirms HTTP/2 is supported, so the **single-packet attack** is the best option. We send the coupon redemption request to Turbo Intruder, strip out some unnecessary headers to keep each HTTP/2 frame lean, and configure the attack:

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/single-packet-attack-1.png" class="alignnone wp-image-4801 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/single-packet-attack-1.png 1248w, https://www.hackcraft.gr/wp-content/uploads/2026/04/single-packet-attack-1-300x208.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/single-packet-attack-1-1030x713.png 1030w, https://www.hackcraft.gr/wp-content/uploads/2026/04/single-packet-attack-1-768x532.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/single-packet-attack-1-705x488.png 705w" sizes="auto, (max-width: 1248px) 100vw, 1248px" width="1248" height="864" alt="race conditions" />

After successfully racing 20 concurrent coupon redemption requests, the discount stacked multiple times, bringing the total price down to \$15.40.

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/spa-poc-1.png" class="alignnone wp-image-4805 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/spa-poc-1.png 1248w, https://www.hackcraft.gr/wp-content/uploads/2026/04/spa-poc-1-300x208.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/spa-poc-1-1030x713.png 1030w, https://www.hackcraft.gr/wp-content/uploads/2026/04/spa-poc-1-768x532.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/spa-poc-1-705x488.png 705w" sizes="auto, (max-width: 1248px) 100vw, 1248px" width="1248" height="864" alt="race conditions" />

If HTTP/2 is not supported, last-byte synchronization over HTTP/1.1 is the fallback. It doesn't eliminate jitter entirely like the single-packet attack, but it reduces it from milliseconds to microseconds, enough for most race windows. For HTTP/1.1, we switch to Engine.THREADED, which manages individual threads per connection and is designed for this use case. In this particular lab, the server supports both HTTP/1.1 and HTTP/2. Using Engine.BURP2 here would negotiate HTTP/2 and multiplex all requests over a single connection, which defeats the purpose since last-byte synchronization relies on separate connections per request.

It's worth noting that this particular lab was designed with a race window so narrow that exploitation is near-impossible without the single-packet attack. A few extra requests may slip through with last-byte synchronization, but not nearly enough to solve the lab. The technique is demonstrated below for completeness, as it remains the go-to approach when HTTP/2 is unavailable.

**Last-Byte Synchronization Forcing HTTP/1.1**

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-sync-1.png" class="alignnone wp-image-4806 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-sync-1.png 1248w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-sync-1-300x208.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-sync-1-1030x713.png 1030w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-sync-1-768x532.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-sync-1-705x488.png 705w" sizes="auto, (max-width: 1248px) 100vw, 1248px" width="1248" height="864" alt="race conditions" />

Achieved result:

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-poc-1.png" class="alignnone wp-image-4807 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-poc-1.png 1248w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-poc-1-300x208.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-poc-1-1030x713.png 1030w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-poc-1-768x532.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/last-byte-poc-1-705x488.png 705w" sizes="auto, (max-width: 1248px) 100vw, 1248px" width="1248" height="864" alt="race conditions" />

## Multi-Endpoint Race Conditions

Multi-endpoint race conditions are not as widely discussed because they often emerge from business logic and multi-step application flows, making them highly context-dependent. The core idea is to identify hidden sub-states that exist between the moment an action (read, write, delete, etc.) is initiated and when it is actually committed on the backend. This takes considerable manual analysis upfront, and because race windows can be extremely narrow, landing a successful attack often involves a fair amount of trial and error.

Before diving into the exploitation part, we must first understand the thought-process behind this type of vulnerability. The thought-process boils down to asking ourselves one important question:

- In the endpoint/functionality I am testing, is there any potential for collisions? Meaning, is there a possibility that two different operations/functions operate on the same object/record?

To answer this, we need to evaluate three things:

1.  **How is the state stored?**\
    If the state lives server-side in a persistent data structure like a database, collision potential is high. If the state is entirely client-side (e.g., a JWT), there is nothing to collide on, and the endpoint can be skipped.
2.  **What type of operation is targeted?**\
    Operations that modify existing data (e.g., changing our account's email) are far more likely to produce exploitable collisions than operations that simply add new data (e.g., adding an additional email address). The latter is primarily relevant for limit-overrun attacks.
3.  **What is the operation keyed on?**\
    For a collision to occur, two concurrent operations/functions need to act on the same record. Identifying the key (e.g., *sessionid*, *userid*, *reset_token*) tells us whether two requests will actually compete over the same resource or just operate on separate records in parallel.

If after evaluating these three points the collision potential remains high, the functionality is worth testing. Some common flows where this collision potential frequently exists:

- Email activation flows
- Invitation acceptance flows
- Multi-step flows in e-shops, such as the example we will cover below
- Multi-step flows where each step is validated server-side

## 

## Exploiting a Multi-Endpoint Race Condition

We will walk through PortSwigger's dedicated lab: **[multi-endpoint race condition](https://portswigger.net/web-security/race-conditions/lab-race-conditions-multi-endpoint)**.\
The scenario models an e-shop where a user adds items to cart, proceeds to checkout if the account balance is sufficient, and finally, the order is completed.

The goal of this lab is to identify and exploit a hidden sub-state, the brief window between when the order is validated and when the order is actually completed. During this window, the cart remains mutable, meaning we can add an extra item to our cart after the balance check has already passed and the order is validated, and have it included in the finalized order. The following diagram illustrates the vulnerability:

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/Multi-step-race.drawio.png" class="alignnone wp-image-4793 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/Multi-step-race.drawio.png 656w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Multi-step-race.drawio-300x49.png 300w" sizes="auto, (max-width: 656px) 100vw, 656px" width="656" height="108" alt="race conditions" />

By doing some preliminary recon on our lab and completing a purchase flow like a regular user, we conclude the following:

- **How is the state stored?** The state is handled server-side through a cookie named session, likely backed by a persistent data structure like a database.
- **What type of operation is targeted?** Adding an item to the cart adds/appends to the cart object, while the checkout endpoint probably reads the current cart state to validate the balance. If the cart remains mutable while the checkout is being processed, the append can slip in after the validation has already passed.

**What is the operation keyed on?** Both the add-to-cart and the checkout actions operate on the same session. Since they share the same key, a collision between them is possible. Based on the above observations, we want to achieve the following: when an item in our cart is affordable (e.g., the \$10 gift card), trigger checkout and add the L33t Leather Jacket to our cart simultaneously. Since we are utilizing the single packet attack here, requests will arrive at the same time, and it is up to the server to decide the processing order.

*The PortSwigger lab solution suggests investigating timing inconsistencies by grouping requests and sending them in sequence over a single connection, prepending a GET request to warm the connection. This is the correct approach for multi-endpoint race conditions — the entire methodology is built around careful benchmarking to determine whether timing delays originate from the backend network architecture or from actual endpoint processing time. Connection warming refers to the observation that the first request sent over a new or idle backend connection typically takes longer due to backend network initialization overhead, not because of the endpoint's actual processing time.*

However, during testing, the sequential request grouping in Repeater did not produce sufficiently consistent timing to benchmark reliably. What did work was using Turbo Intruder with the single-packet attack directly. The steps to reproduce are as follows: add the gift card to the cart manually through the browser, then immediately fire the Turbo Intruder script, which sends a checkout and two add-to-cart requests for the L33t Leather Jacket simultaneously.

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/multi-step-req-2.png" class="alignnone wp-image-4803 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/multi-step-req-2.png 1331w, https://www.hackcraft.gr/wp-content/uploads/2026/04/multi-step-req-2-300x193.png 300w, https://www.hackcraft.gr/wp-content/uploads/2026/04/multi-step-req-2-1030x662.png 1030w, https://www.hackcraft.gr/wp-content/uploads/2026/04/multi-step-req-2-768x493.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/multi-step-req-2-705x453.png 705w" sizes="auto, (max-width: 1331px) 100vw, 1331px" width="1331" height="855" alt="race conditions" />

After some attempts, the race condition was successfully exploited. The order was completed with both the gift card and the L33t Leather Jacket included, despite only having \$10 in store credit. The resulting store credit of -\$1,267.00 confirms the balance check was bypassed during the race window.

<img src="https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02.png" class="alignnone wp-image-4809 size-full" loading="lazy" decoding="async" srcset="https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02.png 1553w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02-286x300.png 286w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02-981x1030.png 981w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02-768x806.png 768w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02-1463x1536.png 1463w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02-1429x1500.png 1429w, https://www.hackcraft.gr/wp-content/uploads/2026/04/Screenshot-2026-04-07-at-15.43.02-672x705.png 672w" sizes="auto, (max-width: 1553px) 100vw, 1553px" width="1553" height="1630" alt="race conditions" />

Some notes on the script: we specify the endpoint and target host manually since we are queueing multiple distinct requests. The *requestsPerConnection* parameter should match the total number of gated requests, in  the case 3. The order of the requests does not matter: whether checkout is queued first or last is irrelevant, since all requests are released simultaneously via the single-packet attack, and it is up to the server to decide the processing order. We include two add-to-cart requests for the L33t Leather Jacket instead of one to increase the probability that at least one lands inside the race window between validation and order completion. If the first add-to-cart loses the race, the second gets another chance to slip in before the order is finalized.

**Security Testing Tip**

The constraint with this method is the packet size. Each HTTP/2 frame carries overhead (9-byte frame header + compressed headers), so the number of requests that fit in a single TCP segment depends on how lean each request is. It is recommended to strip everything that doesn't affect server-side processing: *Accept,* *Accept-Language,* *Accept-Encoding,* *User-Agent*, and any other default headers. With minimal headers, a short POST body and a session cookie, 20 to 30 requests per packet can be realistic. As a sidenote, *Content-Length* does not need to be set manually, most HTTP/2 tooling handles this automatically. When the target supports HTTP/2, this is always the preferred technique.

## Recommendations

Race conditions are architectural issues, not implementation bugs in the traditional sense. Patching them requires ensuring that security-critical operations cannot be subverted by concurrent requests. The following strategies should be considered (where applicable):

- **Use atomic operations.** Combine the check and the action into a single indivisible operation. In database terms, this means using constructs like UPDATE ... WHERE balance \>= price instead of performing a separate SELECT to check the balance and then a subsequent UPDATE to deduct it.
- **Apply pessimistic locking.** Use database-level locks, such as SELECT ... FOR UPDATE to ensure that once a resource is being read for a decision, no other transaction can modify it until the current one is completed. This serializes access to the shared resource and eliminates the race window.
- **Apply optimistic locking.** Assign a version number or timestamp to the resource. Before committing a change, verify the version has not changed since the initial read. If it has, reject the operation and force a retry. This avoids holding locks but handles contention at commit time.
- **Enforce idempotency.** For operations that should only execute once, such as payment processing or coupon redemption, enforce a unique idempotency key per transaction. If a duplicate request arrives, the server recognizes it and discards it.
- **Serialize critical sections.** For multi-step workflows where atomicity is not achievable at the database level, enforce sequential processing of requests that operate on the same resource through application-level mutexes or queues.
- **Keep session and ORM operations internally consistent.** Session handlers and ORMs that update variables individually rather than as a batch create exploitable sub-states between writes. If a framework abstracts away transactions, it assumes full responsibility for handling them safely. The abstraction alone does not eliminate the concurrency risk.
- **Do not rely on one storage layer to secure another.** Sessions are not suitable for preventing race conditions that occur at the database layer. Each layer must enforce its own concurrency guarantees independently.
- **Consider eliminating server-side state where appropriate.** In some architectures, pushing state client-side using encryption (e.g., signed JWTs) can remove server-side sub-states entirely. This eliminates the shared resource that concurrent requests compete over, though it introduces its own trade-offs around token revocation and payload size.
