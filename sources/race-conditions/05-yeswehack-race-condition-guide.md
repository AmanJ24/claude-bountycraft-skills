---
title: "Concurrency exploits: The ultimate Bug Bounty guide to exploiting race condition vulnerabilities in web applications"
author: Alex Brumen (YesWeHack)
url: https://www.yeswehack.com/learn-bug-bounty/ultimate-guide-race-condition-vulnerabilities
date_published: 2025-09-26
date_scraped: 2026-11-17
platform: other
disclosed_report: none
bounty: not applicable (research/reference)
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

Imagine snapping up a \$1,000 gadget for just \$12 simply by triggering a race condition in the checkout flow.

Race condition vulnerabilities often lead to severe impacts – such as bypassing business logic, escalating privileges or stealing funds – that code reviews and automated scans readily overlook. With distributed systems and async frameworks making shared-state interactions increasingly complex and error-prone, race-condition exploitation is a skill well worth learning for Bug Bounty hunters.

In this guide, you'll learn how attackers exploit concurrency flaws, from last-byte synchronisation to single-packet attacks, and how to use Burp Suite's extension Turbo Intruder. You'll learn how race condition attacks work, understand the root causes of these timing bugs, and concrete strategies to bulletproof web applications against them.

## What are race condition vulnerabilities?

A race condition vulnerability arises when multiple threads or processes concurrently access and modify shared resources without proper synchronisation, leading to unpredictable and potentially erroneous outcomes.

The final state depends on the order and timing of the concurrent operations – effectively creating a ‘race’ to modify the resource. This lack of controlled access can result in data corruption, inconsistent state, denial-of-service or privilege escalation, depending on the nature of the shared resource and the operations performed.

To illustrate a race condition, consider the following Go code snippet:

1package main

2\

3import (

4    "fmt"

5    "sync"

6)

7\

8var counter int = 0

9var wg sync.WaitGroup

10\

11func increment() {

12    for i := 0; i \< 10000; i++ {

13        counter++

14    }

15    wg.Done()

16}

17\

18func main() {

19    wg.Add(2)

20    go increment()

21    go increment()

22    wg.Wait()

23    fmt.Println("Final counter value:", counter)

24}

This code demonstrates a race condition where two goroutines run concurrently and increment a shared counter variable without synchronisation. The `counter++` operation is non-atomic, leading to lost updates. This most likely results in an incorrect counter result.

Web applications rely heavily on threads to handle multiple users concurrently. When multiple HTTP requests arrive at a web server, each request is typically assigned to a separate thread. This allows the server to handle each HTTP request individually and asynchronously.

When multiple HTTP requests are handled asynchronously and interact with shared data, such as during database updates or token generation, race conditions can easily occur. The risk is highest when the changes made by one request aren’t properly synchronised with other requests. This lack of synchronisation between threads can lead to inconsistent or incorrect data.

**ALSO IN THIS SERIES:** [The ultimate Bug Bounty guide to exploiting SQL injection vulnerabilities](https://www.yeswehack.com/learn-bug-bounty/vulnerability-vectors-sql-injection)

## Race condition attack vectors in web applications

Race conditions are a productive avenue for Bug Bounty hunters since they slip under the radar of conventional vulnerability scanners. This is because unearthing these vulnerabilities typically require a deep understanding of the target application’s inner workings.

Unlike easier-to-spot vulnerabilities such as cross-site scripting (XSS) bugs, a successful race condition attack may require numerous exploit attempts. This is because probabilistic exploitation must be timed precisely relative to the server's processing sequence.

Although some race conditions can be exploited deterministically with the right setup, exploitation is often a trial-and-error process.

Let's explore some specific ways race conditions can be exploited. Race conditions have been around for a long time and can occur almost anywhere in a system where multiple components access shared resources. This is because modern systems often run programs that handle multiple processes asynchronously or in parallel – opening doors to potential race condition flaws.

### HTTP/1.1 last-byte synchronisation attacks

The most common attack technique is last-byte synchronisation, which abuses how HTTP/1.1 servers handle requests and responses. By keeping the first request open for the precise time window needed, you force the server to juggle two partially processed operations.

Last-byte synchronisation exploits this overlap to bypass security checks. Essentially, an attacker attempts to send two requests to the server almost simultaneously, carefully timing them so the server begins processing the second request while still handling the tail end of the first. The aim is to exploit this brief overlap in processing to trigger a race condition.

For example: If the first request is crafted to keep the connection open – while expecting further data – and the second request is timed perfectly, it can lead to bypassed security checks or access to restricted resources.

### HTTP/2 single-packet race condition attacks

Despite being a more modern protocol, HTTP/2 still has its own exploitable quirks. [Multiplexing](https://datatracker.ietf.org/doc/html/rfc9113#name-streams-and-multiplexing) makes HTTP/2 faster, but also creates unexpected timing gaps. Holding back fragments of each request lets you bundle them into a single packet – neatly bypassing network jitter and improving the reliability of race condition testing.

The trick here involves sending multiple HTTP requests over the same HTTP/2 connection. We deliberately hold back a small portion of each request just long enough to prevent the server from processing them immediately. Then, after a brief pause, we send the final fragment of each request. This ensures all requests are transmitted within a single TCP packet to the server, eliminating network jitter delays.

For more in-depth insights on how single-packet attacks work, I recommend reading James Kettle's fantastic article entitled [‘The single-packet attack: making remote race-conditions “local”’](https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local).

## Hands-on Lab: exploiting race conditions with Burp Suite Turbo Intruder

[Turbo Intruder](https://github.com/PortSwigger/turbo-intruder) is one of the best tools for detecting and exploiting race condition vulnerabilities. This Burp Suite extension supports both last-byte sync and single-packet attacks, and can easily be customised for novel attacks due to its Jython scripting engine.

Now we’re familiar with the theoretical aspects race condition attacks, let’s put our practical skills to the test.

The PortSwigger lab called [‘limit overrun race conditions’](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun) challenges us to exploit a business logic flaw with a race condition. The vulnerability lies in how the coupon code is validated, allowing us to apply the discount multiple times. The goal is to buy a ‘Lightweight L33t Leather Jacket’ priced originally at €1,337 for the sum in our account balance – €50 – or less.

We add the jacket to the cart and apply coupon code PROMO20, but the price – a whopping €1069.60 – is still well beyond our budget.

![Article image](/_next/image?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fd51e1jt0%2Fproduction%2F1b3e288fa9830683c5aad66bbe80bd3456cd3adb-1529x1229.png%3Fw%3D1200%26q%3D85%26auto%3Dformat&w=3840&q=85)

So we intercept the request and send it to Turbo intruder, where we craft and launch [this single-packet attack](https://github.com/PortSwigger/turbo-intruder/blob/master/resources/examples/race-single-packet-attack.py):

1def queueRequests(target, wordlists):

2\

3    # if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack

4    # if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead

5    # for more information, check out https://portswigger.net/research/smashing-the-state-machine

6    engine = RequestEngine(endpoint=target.endpoint,

7                           concurrentConnections=10,

8                           engine=Engine.BURP2

9                           )

10\

11    # the 'gate' argument withholds part of each request until openGate is invoked

12    # if you see a negative timestamp, the server responded before the request was complete

13    for i in range(20):

14        engine.queue(target.req, gate='race1')

15\

16    # once every 'race1' tagged request has been queued

17    # invoke engine.openGate() to send them in sync

18    engine.openGate('race1')

19\

20\

21def handleResponse(req, interesting):

22    table.add(req)

![Article image](/_next/image?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fd51e1jt0%2Fproduction%2Fc0ff33c74d7029114eba2e8ff7caa79c8b89f0cf-2097x1826.png%3Fw%3D1200%26q%3D85%26auto%3Dformat&w=3840&q=85)

The single-packet attack worked like a charm: we got a discount of €1,299.38 and a bargain basement price of €37.62!

![Article image](/_next/image?url=https%3A%2F%2Fcdn.sanity.io%2Fimages%2Fd51e1jt0%2Fproduction%2F215ce7b3100d5a9fab074c9591154609bf4f35a2-1575x1400.png%3Fw%3D1200%26q%3D85%26auto%3Dformat&w=3840&q=85)

## Real-world race condition CVEs

- [**CVE-2022-4037**](https://nvd.nist.gov/vuln/detail/CVE-2022-4037) - A race condition in GitLab CE/EE that can lead to verified email forgery and takeover of third-party accounts when using GitLab as an OAuth provider.
- [**CVE-2016-5195**](https://nvd.nist.gov/vuln/detail/cve-2016-5195) - Race condition in mm/gup.c in the Linux kernel 2.x through 4.x before 4.8.3 allows local users to gain privileges by leveraging incorrect handling of a copy-on-write (aka ‘[Dirty COW](https://en.wikipedia.org/wiki/Dirty_COW)’)
- [**CVE-2024-58248**](https://nvd.nist.gov/vuln/detail/CVE-2024-58248) - nopCommerce before 4.80.0 does not implement locking for order placement, creating a race condition with duplicate redeeming of gift cards.

**YOU MIGHT ALSO LIKE:** [The ultimate guide to Bug Bounty reconnaissance and footprinting](https://www.yeswehack.com/learn-bug-bounty/recon-series-recap-reconnaissance-footprinting)

## Security impacts of race condition vulnerabilities

The impacts of race conditions can manifest in unpredictable ways, making detection and mitigation challenging. These vulnerabilities can lead to:

- **Data corruption:** Concurrent access to shared resources without proper synchronisation can result in data being overwritten or modified inconsistently, leading to data corruption and application malfunction.
- **Business logic flaws:** Race conditions can allow attackers to manipulate business logic, particularly in payment transfers or coupon applications. By rapidly submitting multiple requests before a transaction is completed, they might bypass validation checks and achieve unintended outcomes, such as applying a coupon multiple times or transferring funds improperly.
- **Denial-of-Service (DoS):** By exploiting a race condition, an attacker can cause a system to enter an unstable state, leading to resource exhaustion or system crashes, resulting in a denial-of-service condition.
- **Improper access controls:** Carefully crafted race conditions can bypass authentication or authorisation checks, allowing unauthorised access to protected resources or functionalities. Imagine a multi-step login process, where a race condition could let someone skip a step and gain unauthorised access.

The severity of the impact depends on the specific context of the application and the nature of the shared resource being affected. However, race conditions are often classed as high or critical vulnerabilities due to their potential for causing significant security breaches.

## Preventing race conditions: Mitigation best practices

Race conditions, as we’ve discussed, can introduce unpredictable behaviour to your applications. How do we address these vulnerabilities and ensure our code behaves as expected?

The most common – and often effective – approach is using synchronisation primitives. These mechanisms act like traffic controllers, ensuring that only one thread or process accesses a shared resource at any given moment.

The most common form of synchronisation primitive is the [**lock or mutex (mutual exclusion)**](https://en.wikipedia.org/wiki/Lock_(computer_science)). A mutex functions like a token that only one thread can hold at a time. Before accessing a shared resource, a thread must acquire the mutex. Once finished, it releases the mutex, allowing another thread to acquire it. Go, for instance, provides the sync.Mutex type for this purpose. Here's how you might use a mutex to protect our problematic counter from our earlier Go snippet:

1package main

2\

3import (

4    "fmt"

5    "sync"

6)

7\

8var counter int = 0

9var wg sync.WaitGroup

10var mu sync.Mutex // Our mutex!

11\

12func increment() {

13    for i := 0; i \< 10000; i++ {

14        mu.Lock()   // Acquire the lock before accessing counter

15        counter++

16        mu.Unlock() // Release the lock after accessing counter

17    }

18    wg.Done()

19}

20\

21func main() {

22    wg.Add(2)

23    go increment()

24    go increment()

25    wg.Wait()

26    fmt.Println("Final counter value:", counter)

27}

The same general principle applies across all programming languages: it's best to avoid modifying critical shared variables from multiple threads unless absolutely necessary. Think of it like a busy intersection: the fewer cars trying to cross at once, the less likely you are to have a crash or, in this case, a vulnerability.

Other race-condition mitigations include:

- **Transactional semantics** –preferring ACID transactions when updating related rows; use SELECT ... FOR UPDATE or equivalents for row-level locks
- **Idempotency & optimistic locking** – designing critical operations to be safely repeatable or detect/reject conflicting updates
- **Atomic operations** – combining validation and state update into one server-side step to avoid race windows
- **Distributed locks** – for multi-process or multi-node deployments (with caution necessary around failure modes)
- **Rate limiting** – restricting sensitive operations to reduce attack surface
- **Concurrency testing** – using stress tests, fuzzers, and race detectors (e.g., Go’s tool) in CI and runtime

## Conclusion: outrunning defences, making mayhem with milliseconds

Race condition vulnerabilities are subtle yet potent flaws. Identifying and exploiting them requires an understanding of concurrency, application logic and timing-based attack techniques.

You hopefully now have a grasp of how race conditions arise, common attack techniques such as last-byte synchronisation and single-packet attacks, and how to use Burp Suite's Turbo Intruder to exploit race conditions thanks to our PortSwigger lab walk-through.

Of course, there’s much more still to learn. Because web stacks evolve rapidly – due to trends like HTTP/2/3, serverless and distributed state increasing complexity – bug hunters must continuously adapt their testing techniques and stay abreast of latest best practices.

To that end, we recommend you check out some of the most insightful or innovative writeups on this topic provided in the ‘further reading’ section below!

**ALSO IN THIS SERIES:** [The ultimate Bug Bounty guide to exploiting CSRF vulnerabilities](https://www.yeswehack.com/learn-bug-bounty/ultimate-guide-csrf-vulnerabilities)

## References & further reading

- ‘[Smashing the State Machine: The True Potential of Web Race Conditions](https://portswigger.net/research/smashing-the-state-machine)’ – James Kettle, PortSwigger
- [The single-packet attack: making remote race-conditions 'local'](https://portswigger.net/research/the-single-packet-attack-making-remote-race-conditions-local) – James Kettle, PortSwigger
- ‘[A Race to the Bottom - Database Transactions Undermining Your AppSec](https://blog.doyensec.com/2024/07/11/database-race-conditions.html)’ – Viktor Chuchurski
- ‘[Beyond the Limit: Expanding single-packet race condition with a first sequence sync for breaking the 65,535 byte limit](https://flatt.tech/research/posts/beyond-the-limit-expanding-single-packet-race-condition-with-first-sequence-sync/)’ – RyotaK
- [Dive into single-packet attacks](https://infosecwriteups.com/dive-into-single-packet-attack-3d3849ffe1d2) – Amin Nasiri
- [PortSwigger Lab: limit overrun race conditions](https://portswigger.net/web-security/race-conditions/lab-race-conditions-limit-overrun)

**HANDS-ON HACKING TRAINING** *Tackle labs and CTF challenges around common vulnerabilities on [DOJO](https://dojo-yeswehack.com), our CTF training platform for bug hunters*
