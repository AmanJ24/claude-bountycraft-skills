---
title: "How To Find Broken Access Control Vulnerabilities in the Wild"
author: Luke Stephens (Hakluke), Founder & CEO @ Haksec
url: https://www.hackerone.com/blog/how-find-broken-access-control-vulnerabilities-wild
date_published: unknown
date_scraped: 2026-11-17
platform: hackerone
disclosed_report: none
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->

# How To Find Broken Access Control Vulnerabilities in the Wild

Luke Stephens, Founder & CEO @ Haksec

Perhaps you've just started bug bounties and you keep hearing people talk about Broken Access Control (BAC) bugs. Or, perhaps you're an experienced hacker who has come here for some more advanced BAC tips and tricks. Either way — this blog has got you covered. We'll start with the basics, so feel free to skip ahead.

## What Is Broken Access Control?

BAC is a class of application vulnerability where a function or asset in the application is accessible to someone who should not have access.

If you're anything like me, that definition will not have gotten you any closer to understanding what BAC means, so here are some fictitious examples of BAC bugs:

### BAC Example 1: An e-commerce website allows unauthorized viewing of customer details

Let's say your favourite online camping shop is having a sales on waterproof hiking boots, so you decide to take a look. You recently moved house, so before you make a purchase you want to check if your delivery address is up to date on your customer profile. You login and navigate to your account settings which takes you to the following URL:

/users/482

In this instance, 482 is your user's numeric identifier. It is very common for applications to store objects in a database, referring to them with a unique numeric identifier. The row of the database might look something like this:

**user_id**: 482
**email**: hakluke@wearehackerone.com
**password**: $2y$10$Y71TNx.PAFwFBySCbJ4EO.pL0LF5w84t/PEK00DGcaQ5bJSTkUjCK
**address**: 955 Broadway, Boston, MA, USA

When navigating to /users/482 the email, address, and credit number are shown on the page.

But, what happens if we change 482 to another number, such as 481?

You navigate to /users/481 and the details of another user are displayed. Of course, these details should _not_ be viewable to you, which means that you have just stumbled across a BAC bug. This could be further exploited by accessing all of the user details from 0-481, which is exactly how some data breaches occur.

### BAC Example 2: A bank allows transactions from unauthorized parties

In this example, we'll cover a slightly different style of BAC vulnerability. Instead of having unauthorized access to sensitive data, the attacker will have unauthorized access to _functionality_.

Let's say that you log into your bank account's website to pay some money to a friend. You notice that upon sending the transaction, your browser sends a request that looks like this:

```
POST /api/v1/transfer HTTP/1.1
Host: bank.example.com
Content-Type: application/json
Authorization: Bearer your_access_token

{
    "from_account": "472938294",
    "to_account": "483928403",
    "amount": 20.00,
    "currency": "USD",
    "transfer_description": "Thanks for the pizza!"
}
```

Your hacker senses start tingling, so you change the "from_account" to 470000000 and the "to_account" to your own bank account id. You forward the request through and $20 instantly appears in your account.

You've found a BAC bug, and made $20!

Just a timely reminder: don't run tests like this unless you have explicit permission because even if you have the best intentions, it may end up landing you in legal trouble.

## What's the Difference Between BAC and IDOR?

Simply put, Insecure Direct Object Reference (IDOR) bugs are a subset of BAC bugs. IDOR bugs refer to a type of BAC bug where the attacker can access an object that they shouldn't be able to access by simply referencing its ID directly. Some examples of BAC bugs that are _not_ IDOR bugs include:

- Privilege escalation through forced browsing (e.g. gaining administrative privileges by simply navigating to /admin)
- The ability to access another user's resources by tampering with a request (for example, saying that your username is the victim's username), but not actually directly accessing an object through its identifier.
- Broken access controls through HTTP verb tampering: e.g. an attacker can update records that they shouldn't be able to update by sending a POST request instead of a GET request in the HTTP request.

## Prevalence of Modern BAC Bugs

It's tempting to think that BAC bugs should be mostly eradicated by now. They're not. In fact, they've recently taken center stage as they've become number one in the OWASP top ten list. There are some very successful bug bounty hunters who focus almost exclusively on these types of bugs.

We've seen a drop in the amount of injection bugs (such as SQL injection) over the last 10 years or so because these types of bugs are able to be solved at a high level by using frameworks that encourage developers to code things securely by default. This is very difficult to do with BAC bugs because permission structures in applications are often complex and application-specific. Access controls usually need to be implemented as custom rules, and defined per function or object. If any are missed - it is likely to result in a vulnerability.

## Types of BAC Identifiers

Specifically for IDOR vulnerabilities, it's important to be aware of the common types of things that developers use as identifiers so that you can spot them quickly when you're scrolling through proxy logs.

### User-Chosen Identifiers

Perhaps the simplest, these are types of identifiers that the user has chosen, such as a username, email address, or URL slug.

### Natural Keys

Natural keys are types of identifiers that naturally occur in the data. For example, when referring to a US citizen you might use their social security number as an identifier.

### Composite Keys

These keys are made up of multiple fields from the object, for example in a standard e-commerce application, you might combine a user identifier with a product identifier to create the identifier of an order.

### Numeric identifiers

The examples given above are numeric identifiers. This is the most common type of identifier, it simply assigns a number to every new object that is created, incrementing by one each time.

For example, in a web application that generates invoices, the first invoice might be accessible at:

/invoices/1

Likewise, the 432nd invoice might be accessible at:

/invoices/432

These identifiers are the easiest to exploit because you can simply increment or decrement the counter to access other invoices.

### UUIDs

A UUID (Universally Unique Identifier) is a 128-bit number that is also often used as an identifier. The probability of generating the same UUID more than once is extremely low. For this reason, they are commonly used in software development for identifying resources such as database entries, objects in distributed systems, or unique keys in databases.

UUIDs are typically represented as a string of hexadecimal digits, divided into five groups separated by hyphens, like this:

550e8400-e29b-41d4-a716-446655440000

In a nutshell, it's extremely difficult (verging on impossible) to exploit broken access controls if UUIDs are being used as identifiers, unless you can leak the identifiers somehow (which is sometimes possible through normal use of the application).

### Hashes

Hashes are often used as identifiers, especially when dealing with files. The file is hashed and then this hash becomes the identifier for that file.

## Techniques for Finding BAC Bugs

There are many ways to find BAC bugs, but here are a few common techniques.

### Permissions Mapping

Once you've chosen your target, create two lists. On the left, create a list of user roles. On the right, create a list of actions that can be performed in the application. For example, for a simple blog application, it may look like this:

| User Roles | Actions |
|---|---|
| Unauthenticated | Create a new blog |
| Editor | Read blogs |
| Administrator | Update a blog post |
| | Delete a blog post |

Now, use a spreadsheet to assign every action to every user, and two more columns. One that says "Should have permission", and one that says "Does have permission". Go through each row and update "should have permission" to "yes" or "no" based on whether that role should have that permission:

| User | Action | Should have access | Does have access |
|---|---|---|---|
| Unauthenticated | Create a new blog | No | |
| Unauthenticated | Read blogs | Yes | |
| Unauthenticated | Update a blog post | No | |
| Unauthenticated | Delete a blog post | No | |
| Editor | Create a new blog | No | |
| Editor | Read blogs | Yes | |
| Editor | Update a blog post | Yes | |
| Editor | Delete a blog post | No | |
| Administrator | Create a new blog | Yes | |
| Administrator | Read blogs | Yes | |
| Administrator | Update a blog post | Yes | |
| Administrator | Delete a blog post | Yes | |

Now the fun begins! For every "No" in the "Should have access" column, test the application to see if you can perform the action. At the end, if there are any rows that have a "No" in the "Should have access" column and "Yes" in the "Does have access" column, you've found a BAC bug. For example:

| User | Action | Should have access | Does have access |
|---|---|---|---|
| Unauthenticated | Create a new blog | No | No |
| Unauthenticated | Read blogs | Yes | Yes |
| Unauthenticated | Update a blog post | No | No |
| Unauthenticated | Delete a blog post | No | No |
| Editor | Create a new blog | No | No |
| Editor | Read blogs | Yes | Yes |
| Editor | Update a blog post | Yes | Yes |
| Editor | Delete a blog post | No | **Yes** |
| Administrator | Create a new blog | Yes | Yes |
| Administrator | Read blogs | Yes | Yes |
| Administrator | Update a blog post | Yes | Yes |
| Administrator | Delete a blog post | Yes | Yes |

In this table, you can see that an "Editor" should not be able to delete a blog post, but they can. Time to get reporting!

For a larger, more complex application, permission mapping can be extremely repetitive and tedious, but having the patience to be comprehensive and thorough pays dividends.

### Autorize

Autorize is a Burp Suite BApp that helps uncover BAC bugs by replaying your normal browsing requests as another user. It then compares the response of the legitimate request to the response of the illegitimate request to determine whether the request was successful or not. It will then color code responses based on how similar they are, allowing you to easily filter out interesting requests. [Learn more about Autorize.](https://portswigger.net/bappstore/f9bbac8c4acf4aefa4d7dc92a991af2f)

## Conclusion

It's official: Broken Access Control is the number one most common security issue found in applications according to OWASP. In my own experience hunting on programs, I can confirm that this is indeed the case. They are also quite a simple bug to understand, meaning that they're a great first bug class to learn about. I hope that this blog post has taught you about BAC bugs and inspired you to go out and find a few of your own.

Happy hacking!

## About the Author

Luke Stephens (known online as Hakluke) is an Australian cybersecurity professional, entrepreneur, and content strategist. He's the founder of Haksec Group, a group of cybersecurity-related companies including Ironbark Cyber, HackerContent and Triagers.
