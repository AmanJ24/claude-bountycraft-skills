---
title: "IDOR and Broken Access Control Risking Private Data Exposure"
author: Ahmed Qaramany (c0nqr0r)
url: https://c0nqr0r.medium.com/idor-and-broken-access-control-risking-private-data-exposure-dd808412ed13
date_published: 2024-10-13
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR and Broken Access Control Risking Private Data Exposure

Hello, guys! Hope you're all doing well. It's Ahmed Qaramany @c0nqr0r here, and I'm excited to be back logging some fresh write-ups. I hope you find something useful and enjoy the content! Let's dive in!

**Lab Setup:**

When testing for **Insecure Direct Object Reference (IDOR)** vulnerabilities, I use a combination of tools to ensure I don't miss any issues. Here's how I approach it:

**Setting Up Firefox and Pwnfox:**

- I use **Firefox** as my browser, with the **Pwnfox extension** installed. Pwnfox allows me to isolate and manage different browsing contexts, making it easier to switch between test accounts.
- I connect Firefox to **BurpSuite**, which captures and monitors all the web traffic between my browser and the application I'm testing. This setup helps me inspect requests and responses in detail.

**Using the Autorize Tool in the Background:**

- At the same time, I run the **Autorize** tool in the background. This tool continuously checks for any missing access control issues that I might overlook while manually testing. Autorize is particularly helpful because it flags any unauthorized access it detects, even after I've moved on from a request.
- I manually check the app through **BurpSuite's HTTP History tab**, which shows all the requests and responses that have passed through the proxy.

**Tracking IDs and Objects:**

- During the testing process, I make sure to check every part of the application for **IDs and objects** (like post IDs, user IDs, etc.), as these are often the key to finding IDOR vulnerabilities.
- To stay organized, I store all the IDs and objects I find in an app called **Obsidian**. Obsidian helps me keep detailed notes about my findings in a structured way.
- I create a specific file in Obsidian for each application I'm testing, where I store important details like the IDs for two test accounts I'm using, as well as any cookies or **JWT tokens** if the app uses them for authentication.

**Testing Features:**

While using the application, I test it thoroughly for **IDOR vulnerabilities**. This means trying out all possible ways a user might access or manipulate resources they shouldn't have permission to touch.

For example, the application might allow users to perform **CRUD operations** (Create, Read, Update, Delete) on posts. I test each of these operations to see if an unauthorized user can exploit them:

- **Create:** Can I create a post as another user?
- **Read:** Can I read another user's post by modifying the post ID?
- **Update:** Can I change the content of a post that doesn't belong to me?
- **Delete:** Can I delete someone else's post?

Even though the app seems to have some access control in place, it wasn't well-implemented across all features. This lack of proper access control allowed me to find a vulnerability related to generating reports for posts.

During testing, I noticed that the platform allowed users to **Create, Read, Update, and Delete (CRUD)** posts. Naturally, I tested all of these operations for IDOR vulnerabilities, focusing on whether the access controls were properly implemented. Unfortunately, it became clear that access control wasn't well enforced across the board.

One feature I discovered was the ability to generate reports for all the posts. The first thought that crossed my mind was, **"Can I access or export reports for other users' posts?"**

I tried manipulating the request by changing the post ID, hoping to send someone else's report to my email. However, the system was secure in this case; the report was sent directly to the email associated with the account, making it difficult to hijack reports from other users by simply modifying the ID.

**After thoroughly clicking through all the buttons and testing each function of the app**, I stumbled upon another export feature. This time, it allowed reports to be exported as PDFs. That's when I realized there might be another opportunity for IDOR exploitation.

At first, I wasn't planning on testing the export feature, but out of curiosity, I clicked the button to download the PDF. After downloading it, I decided to check what was happening behind the scenes. So, I opened **BurpSuite** and navigated to the **HTTP history** tab to see the requests that had been made.

This was the key moment that led me to investigate further. By reviewing the HTTP requests, I could see how the app handled the PDF generation and the associated parameters. This gave me the opportunity to test whether I could manipulate the request, particularly the **post ID**, to see if I could download someone else's report.

Feel that? Those last two requests are perfect for testing IDOR. Let's send them to Repeater and see what we find!

In Request (1), I found two interesting IDs with a **CREATED** status:

```
"backgroundJobId":"d69047b7-3a7a-4784-96be-6633b6d8bd45","status":"CREATED"
```

This ID caught my attention, so I stored it in **Obsidian** for reference.

Then I opened the other request and found there's more to explore...

In **Request (2)**, I noticed it was using the same **backgroundJobId** that I had already stored in my **Obsidian** notes. The response contained the URL to download the content for just **one single post** (not all my posts). This got me thinking: what if I could access other people's reports?

To try this, I went back to the request that returned the **backgroundJobId** and sent it to **BurpSuite's Intruder**. Since the platform uses sequential numbers for IDs, I set up a bruteforce attack with a range from **0 to 9,999,999**. The goal was to retrieve all the valid post IDs by testing each number in sequence.

Once I got the valid **backgroundJobId** values for other users' posts, I manually inserted them into the following URL:

```
https://s3.amazonaws.com/xxxxxcompanyname/report/xxxxxxx/20241013/{backgroundJobId}.pdf
```

By doing this, I could potentially access other users' PDF reports without authorization, exposing their private posts information, i was able to see all the posts of the application

By the way, we actually have **two vulnerabilities** here:

1. **IDOR (Insecure Direct Object Reference):** The first bug is an IDOR vulnerability, where we were able to retrieve the **backgroundJobId** by manipulating post IDs in the request. This allowed us to access other users' reports.
2. **Broken Access Control:** The second issue is a **Broken Access Control** vulnerability. The application relies only on using **UUIDs (Universally Unique Identifiers)** for securing access to reports, but that's not enough. Since there's no additional security layer, like a signature or proper authorization checks, an attacker can still exploit the system by guessing or bruteforcing the UUIDs. This means unauthorized users can access reports simply by knowing or guessing a valid UUID.

In short, the application doesn't implement proper security mechanisms to prevent unauthorized access, making both vulnerabilities dangerous.

(Author note: write-up was drafted with ChatGPT assistance. Author also publishes Arabic videos on access control, IDOR and privilege escalation on YouTube "BugsAR".)
