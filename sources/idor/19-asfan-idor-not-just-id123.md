---
title: "IDOR Is Not Just id=123: A Simple Guide to Smarter IDOR Hunting"
author: Muhammed Asfan | Cybersecurity Researcher
url: https://medium.com/@MuhammedAsfan/idor-is-not-just-id-123-a-simple-guide-to-smarter-idor-hunting-5f58de1109dd
date_published: 2026-01-03
date_scraped: 2026-11-16
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# IDOR Is Not Just `id=123`: A Simple Guide to Smarter IDOR Hunting

When most people hear "IDOR," they picture one thing:

> Change `id=123` to `id=124` and suddenly you see someone else's data.

That's the classic example, but modern IDOR goes much deeper than just incrementing IDs in the URL. Apps hide object references in JSON bodies, filenames, tokens, and background APIs. If you only look for `id=123`, You'll miss a lot of real bugs.

This post is a short, reader-friendly guide to the different logics and patterns behind IDOR so you can hunt it more effectively.

## What IDOR Really Means

IDOR (Insecure Direct Object Reference) happens when an application lets you access or modify a resource just because you know its identifier, without properly checking whether you're allowed to.

So the real question you should always ask is:

> "Is the server actually checking that this object belongs to me or is allowed for my role?"

If the answer is "no" and changing the identifier lets you touch other people's stuff, you're in IDOR territory.

## 1. It's Not Only `id`: Many Things Are "Object References"

The word "object" is important. The app isn't just protecting numbers; it's protecting objects like users, orders, files, tickets, etc. The reference to those objects can be in many formats.

Common patterns:

### Simple numeric IDs

- `GET /orders/531` -> try `532`, `533`, etc.
- `GET /invoice?invoice_id=9001` -> try `9002`.

If changing the number shows another user's order or invoice, that's a straightforward IDOR.

### UUIDs and GUIDs

```
GET /api/v1/files/550e8400-e29b-41d4-a716-446655440000
```

Just because it's not `123` doesn't mean you ignore it. If you find another file ID anywhere (JavaScript, logs, error messages, emails) and can access it with your low-privileged account, that's still an IDOR. The format changed, the logic didn't.

### Tokens, slugs, filenames

Object references can also be:

- Filenames: `GET /download?file=report_q4.pdf`
- Slugs: `GET /blog/my-first-post`
- Tokens: `GET /document?token=ABCD1234`

If swapping those values reveals or modifies resources that belong to other users, you've found the same class of problem: the app trusts the reference, not the user.

### Composite references

```
GET /order?user_id=5&order_id=123
```

If the backend only uses `order_id` to load the order and ignore `user_id`, changing just the `order_id` might let you pull another user's order. Composite references like this are a great place to test whether the developer actually uses all the fields for authorization or just one.

## 2. IDOR Is Not Only About Reading: Actions Matter Too

Many people stop at "I can view another user's data." That is good, but some of the best IDORs are about actions, not just viewing.

Think in CRUD:

### Read IDOR

- View someone else's: Profile, Order, Invoice, Support ticket

### Update IDOR

- Example: `PUT /api/user/123 { "email": "new@mail.com" }`
- If you change `123` to `124` and can update another user's email or settings, that's a more severe IDOR.

### Delete IDOR

- Example: `DELETE /api/posts/88`
- If you can delete posts you don't own by swapping the ID, you've found an authorization flaw with clear impact.

### Action/workflow IDOR

Sometimes the endpoint doesn't obviously say "delete" or "update", but performs powerful actions:

- `POST /admin/ban_user` with `user_id=124`
- `POST /orders/confirm` with `order_id=5002`

If any logged-in user can hit these endpoints and change the IDs to affect other users, this becomes "IDOR in admin actions" or "IDOR in business workflows," which is usually high severity.

## 3. IDOR Hides in JSON, Bodies, and APIs

Modern apps use APIs heavily. IDOR often lives in requests you never see in the URL bar.

### JSON bodies

```
POST /api/user/details
Content-Type: application/json

{
  "user_id": 123
}
```

Try changing `"user_id": 123` to another user's ID and see if you can fetch their details. Don't just focus on query parameters; always inspect JSON payloads in your proxy.

### Nested or complex JSON

```
POST /api/orders/get
Content-Type: application/json

{
  "order": {
    "id": 5001
  }
}
```

IDs can be buried deep. Modify `5001` and check whether the server still returns data with no extra authorization checks.

### Path + body mismatch

- URL: `/api/users/123`
- Body: `{ "id": 124 }`

Developers might check only one of these. Try sending:

- URL with your own ID, body with someone else's ID.
- URL with someone else's ID, body with yours.

You're testing "What does the backend actually trust here?"

### File and export endpoints

- `GET /files/1234/download`
- `GET /export?file_id=789`

Change the IDs or file references and see if you can download other users' exports, reports, backups, or invoices.

## 4. It's Not Always 123 -> 124: Other ID Patterns

### Sparse or non-sequential IDs

Orders might be: `1000`, `2000`, `3000` instead of `1, 2, 3`.

In those cases:

- Try guessing around the IDs you see in responses.
- Try stepping patterns (e.g., +-10, +-100), depending on what the app returns.
- Use error messages and logs to learn which IDs exist.

### Leaked references from the app itself

Often, the app gives you IDs in:

- Emails (order confirmation links)
- JavaScript variables
- HTML comments
- Error messages
- Exported CSV / JSON

If you see `invoice/9009` in your own data and later see `invoice/9042` mentioned somewhere else, test whether you can fetch `9042` too. That's still IDOR, even though you didn't "guess" it by brute forcing.

### Role confusion

Sometimes the same endpoint is used for both normal users and admins, and the only thing that should separate them is an authorization check.

If your low-privileged user can:

- Call an endpoint meant for admins
- Change IDs to affect data that should be admin-only

Then you've essentially found IDOR at the role/authorization layer. The app is trusting "you know this reference" instead of "you have this permission".

## 5. A Tiny IDOR Checklist You Can Reuse

**Spotting candidate parameters**

- Does this parameter look like an object reference?
- Number, UUID, token, filename, slug, or nested ID in JSON.

**Basic tampering**

- What happens if I change it to:
  - A higher or lower value?
  - An ID I saw in another response, email, or JS file?

**Testing operations**

- After changing the ID, can I:
  - View data belonging to another user?
  - Modify someone else's data (email, address, role, settings)?
  - Delete their resources (posts, files, tickets)?
  - Trigger actions like approve, confirm, cancel, or ban for objects I don't own?

**Authorization thinking**

- Is the server clearly checking:
  - "Does this object belong to this logged-in user?"
  - "Does this role have permission to access this object?"
  - Or is it just: "If you send a valid ID, I'll do it"?

If the backend is acting blindly on the ID you send, you've likely moved beyond the simple `id=123` trick and into real broken object-level authorization.

You can keep this post as your mental model: next time you see any `id`, UUID, or token in a request, don't just think "123 -> 124". Think "What object does this represent, and what happens if I point it at someone else's object instead?"

Tags: Idor, Cybersecurity, Bug Bounty Tips, Bug Bounty Writeup, Broken Access Control
