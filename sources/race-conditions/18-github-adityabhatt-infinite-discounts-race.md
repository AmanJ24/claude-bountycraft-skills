---
title: "Bug Bounty Race: Exploiting Race Conditions for Infinite Discounts"
author: AdityaBhatt3010
url: https://raw.githubusercontent.com/AdityaBhatt3010/Bug-Bounty-Race-Exploiting-Race-Conditions-for-Infinite-Discounts/main/README.md
date_published: unknown
date_scraped: 2026-11-17
platform: github
disclosed_report: none
bounty: not stated
vuln_class: race-conditions
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

# Bug Bounty Race: Exploiting Race Conditions for Infinite Discounts

Race Condition Vulnerability hands on experience to apply discount coupons multiple times and bypass intended pricing logic.

(Note: this is a PortSwigger Academy lab write-up (Limit Overrun, Apprentice), not a disclosed bug bounty report.)

---

## What is a Race Condition?

A **Race Condition** occurs when two or more operations are executed concurrently, and the final outcome depends on the **timing or order** of execution. If these operations are **not synchronized properly**, it can lead to unexpected or malicious results.

In simpler terms:

> It's like two people racing to withdraw money from a shared ATM account at the same time, both see the same balance and withdraw it, resulting in **double-spending**.

---

## Why Do Race Conditions Matter in Security?

In secure applications, especially those dealing with:

* **Payment systems**
* **Authentication/authorization**
* **Inventory management**
* **Coupon codes or rewards**

Race conditions can be **exploited** to bypass checks or perform unauthorized actions. These are a class of **business logic vulnerabilities**, flaws in the way the application was designed to handle actions in sequence.

---

## Race Conditions in Bug Hunting

In bug bounty hunting or pentesting, race conditions are a goldmine because they often:

* Lead to **critical financial losses** (e.g., double-purchases with coupons)
* Bypass **authorization or verification**
* Allow **escalation of privileges**
* Cause **data corruption or leakage**

### Common Use Cases in Bug Hunting:

| Scenario                        | Exploit                                                    |
| ------------------------------- | ---------------------------------------------------------- |
| **Multiple Coupon Application** | Apply same coupon simultaneously to get stacked discounts  |
| **Bypassing Payment Gateways**  | Trigger payment confirmation and cancel before debit       |
| **Reset Password Abuse**        | Simultaneously change password and verify token            |
| **Inventory Exploitation**      | Purchase limited-stock items more than allowed             |
| **Privilege Escalation**        | Send simultaneous role change and privilege check requests |

---

## How are Race Conditions Exploited?

Exploitation typically requires sending **parallel or concurrent requests** that manipulate server-side logic before it updates the shared state.

### Tools and Methods:

* **Burp Suite Repeater's Tab Groups**: Sending requests in **parallel**
* **Fuzzing with Intruder or Turbo Intruder**
* **Custom Python/Go scripts** with threading
* **Intercepting sensitive state-changing requests** (like coupon or checkout) and duplicating them

---

## Signs of a Race Condition:

* A **value is updated after a check** is made (TOCTOU: Time of Check to Time of Use).
* Operations lack **atomicity** (not all done in one indivisible step).
* Repeated or concurrent actions produce **inconsistent results**.

---

## Real-World Examples

1. **Tesla Bug Bounty (2020)**
   Researcher purchased vehicle software upgrades for **free** by sending multiple purchase requests simultaneously.

2. **Uber Bug Bounty (2016)**
   Exploited a logic flaw to **earn promo credits infinitely**.

3. **e-Commerce Sites**
   Repeated coupon code application reduced prices to **zero** due to missing locking.

(Author's claims, uncited in the source.)

---

## Connecting to the Lab: Limit Overrun Race Condition (BurpSuite)

### Objective:

Buy a **Lightweight L33t Leather Jacket** for a price lower than intended by **abusing race condition** in coupon application logic.

### Vulnerability:

The app fails to **lock coupon usage** during the **apply coupon** process, allowing **multiple discounts** via simultaneous requests.

### Exploitation Flow:

1. Intercepted the `POST /cart/coupon` request applying the discount.
2. Duplicated the request in Burp Suite **Repeater tab group** (20 times).
3. Sent all requests **in parallel**, hitting the server **simultaneously**.
4. Server responded with **multiple success messages**, applying the discount multiple times.
5. Final cart value = jacket price - (discount x number of successes).
6. Purchased the jacket with **less than original cost**, lab marked **Solved**.

### PoC WriteUp: BurpSuite Lab: Limit Overrun Race Conditions (APPRENTICE)

#### Steps Performed

1. **Accessing the Lab**
   * Log in using `wiener:peter` credentials and accessed the application. [image]

2. **Viewing the Jacket**
   * Navigate to the product catalog and identify the target item: **Lightweight L33t Leather Jacket**. [image]

3. **Adding the Jacket to Cart**
   * Add the jacket to the cart to prepare for the exploit. [image]

4. **Opening the Cart, Applying the Coupon, and Setting Up Proxy**
   * Open the cart, apply the coupon code.
   * Enabl **BurpSuite Proxy** using **FoxyProxy** to begin interception. [image]

5. **Intercepting the Request**
   * Intercept the **POST /cart/coupon** request containing the discount code.
   * Later it can be Dropped and the page refreshed after Step 10. [image]

6. **Repeater Screenshot**
   * Send the coupon application request to **Repeater** for manual testing. [image]

7. **Creating Duplicate Requests**
   * Duplicate the **POST /cart/coupon** request 40 times.
   * Ctrl + R can be used after selecting the request number or right click the request > Duplicate tab > Enter 40. [image]

8. **Adding All Tabs to a Group**
   * Add the original and all duplicate requests into a single **tab group** in Burp Repeater. [image]

9. **Group Screenshot**
   * Capture a screenshot of the tab group showing all 20 prepared requests. [image]

10. **Send Group in Parallel**
    * Send the **tab group in parallel** to exploit the race condition on the server. [image]

11. **Race Condition Applied**
    * Observe that multiple coupon applications were accepted due to lack of proper locking on the server. [image]

12. **Lab Completed**
    * Jacket price reduced drastically; purchased it using store credit and completed the lab successfully. [image]

### Important Note: Discrepancy Handling

In case the race condition doesn't apply correctly during **Step 10-12** (i.e., parallel requests don't result in multiple discounts), simply:

* Remove any applied coupons from the cart.
* Re-execute Steps 10-12.
  This issue may occur due to glitching in parallel request synchronization or server handling.

### InfoSec Insight

This lab is an excellent example of how **lack of concurrency control** on critical endpoints (like discounts, purchases, and authentication) can lead to severe **business logic vulnerabilities**.

### Key Learning Points:

* Use **BurpSuite's Repeater with Parallel Execution** to exploit timing issues.
* Always test endpoints that modify server state (e.g., `POST /cart/coupon`) for **idempotency and synchronization flaws**.
* Real-world impact: Attackers can exploit such flaws for **fraudulent purchases**, **financial theft**, or **DoS attacks**.

---

## Mitigation Techniques (for Developers)

* Use **database-level locking** or atomic operations.
* Implement **idempotency checks** on endpoints like coupon usage.
* Apply **mutexes or semaphores** in critical sections of business logic.
* Detect and block **parallel requests** to sensitive endpoints.
* Add **unique one-time tokens** to enforce single-use operations.

---

## Conclusion

This lab wasn't just about grabbing a discount, it was about understanding how race conditions can break business logic in real-world applications. As bug bounty hunters, it's our responsibility to think beyond the surface and dig deep into concurrency flaws that often go unnoticed. Tools like Burp Suite, paired with precise timing and logic, can turn milliseconds into meaningful impact.
> **Happy Hacking**, and may your payloads always race ahead of the patch!

Capture status: FULL (README text; images not captured).
