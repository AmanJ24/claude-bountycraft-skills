---
title: "Vulnerability disclosure: Authentication bypass in Auth0"
author: Laban Sköllermark (Sentor / Accenture Security)
url: https://sentorsecurity.com/blog/vulnerability-disclosure-authentication-bypass-in-auth0/
date_published: 2023-10-24
date_scraped: 2026-11-17
platform: other
disclosed_report: none
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- Scrape note: the HTTP Request/Response/Decoded JWT code blocks and screenshots were not captured by the scraper (headings remain). Note: despite the filename, the main bug is a cross-tenant session/authorize flaw; the none-alg bypass is mentioned only as prior art. -->

This blog post is written by Laban Sköllermark, pentester and IT security consultant at Sentor, a part of Accenture Security.

This vulnerability is not the same as the session fixation (https://sentorsecurity.com/blog/vulnerability-disclosure-session-fixation-in-auth0/), and in my opinion much more impactful.

At Sentor we often see vulnerabilities in authentication, authorization and session management during pentests of our clients' web applications when they have implemented it themselves. The general recommendation in the industry is to use standard authentication and authorization implementations in frameworks rather than implementing everything yourself. The reason for this is that it is very easy to make mistakes. This is like the popular and widely accepted recommendation "don't roll your own crypto".

But there can be flaws in any system - even the big and widespread solutions for authentication and authorization. There are a lot of terms with abbreviations in this area: Identity Access Management (IAM) / Identity Provider (IdP) / IdP as a Service (IdPaaS) / Identity as a Service (IDaaS).

When I found the session fixation problem in Auth0, I needed to repeat that attack and make sure that I could create a working Proof of Concept (PoC) before reporting it to Auth0. That was when I stumbled upon this, much worse, vulnerability! In certain usage scenarios of Auth0, it lets an attacker impersonate any victim in an Auth0 customer's system, simply by knowing the victim's email address.

Disclaimer: Auth0 have only acknowledged receipt of my vulnerability report and not confirmed my assumptions. They have since then changed the behavior of their system so that this vulnerability is not present anymore. Because of that and due to my 90-day coordinated disclosure deadline has passed, Sentor are now publishing the details.

#### About Auth0 by Okta

If you read the previous post on session fixation and don't need a recap, you can skip this section.

Auth0 is quite well-known in the web application sphere. They started in 2013 and offered a Software as a Service (SaaS) for identity management. They have published a lot of open-source Software Development Kits (SDKs) on GitHub, like the java-jwt library for signing and verifying JSON Web Tokens (JWTs) in Java. They are also the ones behind the site jwt.io, which is mainly for decoding and verifying JWTs. You can also use the site to experiment with signing.

Auth0 was acquired by Okta (the process started in 2021 and was completed in 2022). Their solution, which has been called just "Auth0" previously, is becoming Okta Customer Identity Cloud, but I will use "Auth0" throughout this blog post.

Auth0 can be used for federated login using for instance Microsoft, Google and GitHub. When you set up your tenant, you can let Auth0 be the directory with users and passwords.

The login page above can be customized both in terms of settings and its design. You can redirect your users to log in on your Auth0 tenant at `[example].eu.auth0.com` or you can include the login box in your web application with Auth0's JavaScript. You can also implement your own login logic by using Auth0's API. If the domain presenting the login form is not the Auth0 tenant (`[example].eu.auth0.com`), an Auth0 non-default feature called Cross Origin Authentication must be enabled. Modern browsers do not let you post form data across domains, such as from `app.example.com` to `[example].eu.auth0.com`, but this can be solved by enabling the Auth0 feature Custom Domain which lets you point your subdomain `login.example.com` to Auth0. In this way, your application at `app.example.com` is allowed to post form data to it (`login.example.com`) because it is under the same domain.

#### OAuth 2.0, OIDC, access tokens, ID tokens, JWTs...

There are a lot of terms around modern web-based identity management and many standards to keep track of. This is not the time to properly introduce them all, but let's agree that it's hard to understand all aspects and get everything correct.

One thing that I think that many, me included, have misunderstood is the difference between access tokens and ID tokens. From oauth.net:

- ID tokens are JWTs. Access tokens can be JWTs but may also be a random string.
- ID tokens should never be sent to an API. Access tokens should never be read by the client.

And Microsoft says:

ID tokens shouldn't be used for authorization purposes. Access tokens are used for authorization. The claims provided by ID tokens can be used for UX inside your application, as keys in a database, and providing access to the client application.

Auth0 themselves also make this distinction on several pages in their documentation. A good overview is presented in their article Access Tokens. They use access tokens both in the form of an opaque string, which cannot be validated without calling Auth0's management API, and in the JWT form, where all information needed for most authorization decisions can be included.

When access tokens are on the JWT format they can look very similar to ID tokens.

The vulnerability described in this blog post allowed an attacker to have Auth0 to sign a JWT ID token containing a victim's user address. I never checked JWT-based access tokens when this vulnerability was present in Auth0's systems. At Sentor I have come across many applications which verify JWT ID tokens and trust their contents when making authorization decisions.

For reference, this is how the claims in a typical Auth0 ID token might look like:

#### Finding authentication bypass in Auth0

When I was working on a Proof of Concept attack on the previously disclosed session fixation vulnerability, I created two Auth0 trial tenants. When repeating the attack I stumbled upon this much worse vulnerability - authentication bypass.

Below is the report I sent them, but with a few edits on spelling, formatting and some clarifications. Screenshots of Auth0 configuration pages are included in the blog post to make it easier to follow, especially for people not having set up Auth0 tenants themselves.

Irrelevant data in HTTP requests and responses has been replaced with `[...]` and JSON data has been prettified for readability.

Hi!

My company does not allow bug bounty programs, so I cannot use the Bugcrowd form to report this.

While reproducing the session fixation vulnerability which I recently reported, I stumbled upon a much worse problem. Please confirm both when you have read this (successful decryption) and when you have tried to reproduce.

The technical details are included below. The Vulnerability Rating Taxonomy (VRT) classification is according to Bugcrowd's definitions.

I wish to know if you consider my finding a security vulnerability, whether you intend to patch it and if that can be done within 90 days.

This bug is subject to a 90-day disclosure deadline. After 90 days elapses, the bug report will become visible to the public.

#### Overview

I have managed to get one tenant to issue a JWT for a user which only exists in another tenant. All "public" applications such as Single Page Applications (SPAs), which have public `/authorize` endpoints, are probably vulnerable. Below I have proven the problem between two US-4 tenants in the preview environment. Currently I don't know if the attacker tenant and the victim tenant must be in the same locality, or if the same region (US in this case) is enough. The problem seems to not be reproducible between regions at least (I tried between EU-2 and US-4 with no success). It's also currently not known if the problem exists at all in the production environment. If the tenant `test21-sentorlab.eu.auth0.com` was successfully "migrated" to the production environment after I upgraded it to the "B2C - Essentials" subscription yesterday (2023-JUL-21), I have reproduced the problem between production and preview environments within a locality (EU-2) as well.

The vulnerability exists both after logging in using Universal Login and after using the Cross Origin Authentication (`/co/authenticate`) endpoint.

With the permission of <redacted customer>, I have tried getting a JWT issued for their `login-stage.<redacted>.<tld>` tenant (`<redacted>.edge.tenants.eu.auth0.com`) but failed. If their tenant is located in the EU-1 locality, that could explain the failure. I have created several EU tenants, but all ended up in EU-2. <Redacted person at customer> is on vacation so I don't currently know which locality their tenant lives in, and I haven't found a way to determine that as an attacker, either.

#### Probable root cause

My analysis of the root cause says that, if a session (`sid`) has any tenant information at all in it, the `/authorize` endpoint fails to validate it before issuing tokens. After successfully authenticating as a valid user in an attacker tenant A, the auth0 cookie can be sent to the `/authorize` endpoint of a victim tenant B and that tenant will happily sign a JWT with its correct key just as if a user with the same email address would have existed in that tenant. An attacker needs to:

- Know the domain of a victim tenant
- Know a client ID in the victim tenant (public information in most applications but especially for SPAs)
- Have their own tenant in the same locality
- Know the email address of a victim user, in case a particular victim user needs to be impersonated
- Create a user representing the victim (by email address) in the attacker tenant

#### Restrictions

Only a Username-Password-Authentication connection has been tested.

I am aware of your recommendation to validate the `nonce` claim in JWTs, but I deem that as unfeasible when for instance having an SPA for logging in and then validating the JWTs in an API backend. I don't think that many customers will have their SPA to either ask their API for a nonce or report a nonce to their API before sending the nonce to the Auth0 login flow. This would have been needed for their API to validate the nonce claim in the incoming JWTs later.

If a victim application is validating the `sub` (subject) claim, it will not accept the forged JWT, because the subject value will be that of the attacker user in the attacker tenant.

Also, if the application is checking that the `email_verified` claim is true, ~~it will not accept the forged JWT unless the attacker manages to fool the victim into verifying the account in the victim tenant~~.

Well, that last assumption did not hold. When writing this blog post I discovered that you could set an email address as verified yourself. Your tenant - your choice. You are the admin and manage your own users, after all.

[screenshot: set email as verified]

I also noticed that you could disable the sending of (enabled by default) email address verification email that is otherwise sent as soon as you add a new user to your Username-Password-Authentication database by first changing from the default Auth0 email provider and then disabling the Verification Email (using Link) email template:

[screenshots: use my own email provider; disable verification email]

That way the victim user won't get any warning that you are going to impersonate them.

#### How to reproduce

I have reproduced the problem between two different trial subscriptions both having a tenant each in the US-4 locality. Note that when this blog post was published, the vulnerability could not be reproduced anymore.

After creating a user with the victim's email address in the attacker tenant and then logging in to the attacker tenant, a request to gain a JWT in the correct (attacker) tenant looks like this:

**Request**

**Response**

**Decoded JWT**

(jwt.io links with example JWTs omitted by scraper capture of code blocks; the two example ID tokens were only linked via jwt.io fragments in the original.)

The attacker then repeats the `/authorize` request but replaces the domain and the client ID to the values of the victim tenant:

**Request**

**Response**

**Decoded JWT**

Note that a user with that email address does not exist in the victim tenant.

For the sake of this blog post and to prove the vulnerability after my trial subscriptions have ended and their tenants been deleted, below are the public parts of the Auth0 generated asymmetric keys used to sign the two JWTs above. Well, I could of course have generated the keys myself and faked the whole thing, but I hope Auth0 won't claim that. Attaching the signing certificates will not help since they are self-signed (root CAs - Certificate Authorities). So, make sure to download the `jwks.json` files yourself if they are still online when you read this. :)

#### Recommendations

The following recommendation to mitigate the vulnerability was sent to Auth0:

- Always check which tenant and client ID a session is valid for before trusting any data in it.

#### Similarities with "nOAuth"

Researchers at Descope discovered the so-called nOAuth vulnerability and published a blog post the 20th of June 2023 called nOAuth: How Microsoft OAuth Misconfiguration Can Lead to Full Account Takeover (https://www.descope.com/blog/post/noauth). That vulnerability was in the combination of Microsoft Entra ID (formerly Azure AD) and how some application developers trusted the email claim of Microsoft issued JWTs. Exploiting that vulnerability also involved an attacker-controlled tenant and allowed impersonation of users.

The day before publication of this blog post I went through Okta's different blog platforms (they have many) and found the article Saying "No Thanks" to nOAuth (https://sec.okta.com/articles/2023/08/saying-no-thanks-noauth), published the 5th of August 2023. It's a response to the nOAuth vulnerability and ends with how Okta's both platforms are different - both Workforce Identity Cloud but also Customer Identity Cloud (Auth0). Given that Okta/Auth0 knew about my vulnerability report at the time, and maybe they also already had mitigated it, a few citations are a bit ironic in my opinion:

"By contrast, the Okta Workforce Identity Cloud (WIC) is architected around per-tenant ("Okta Org") federation, and it's up to the Org administrator to determine what identifiers to support. Our tenant boundary is strict: an org administrator can't impersonate users in a different org."

Is it careful wording there? They don't say anything about Customer Identity Cloud.

"While Okta Customer Identity offers the option to allow unverified emails to be used as part of Self-Service Registration (see screenshot below); the blast radius is again squarely within the tenant itself."

Well, they might just have mitigated such a vulnerability...

"Furthermore, the issuer of Microsoft tokens is "MicrosoftOnline", whereas for Okta it is your-org.okta.com."

In this vulnerability it does not have any difference since it's possible to have that issuer to issue tokens with attacker-controlled information and signed with the correct keys of the issuer. But it's good to have several layers of security!

#### Previous Auth0 authentication bypass vulnerabilities

In 2017, the company Cinta Infinita found and reported an authentication bypass vulnerability in the Auth0 platform, which they disclosed publicly a year later in their blog post Knocking Down the Big Door - How We Bypassed the Auth0 Authentication (https://medium.com/@cintainfinita/knocking-down-the-big-door-8e2177f76ea5). On the same day, Auth0 published the security bulletin CVE-2018-6873: Security Vulnerability in the Auth0 Authentication Service and their Chief Security Officer (CSO) wrote about the whole process in the blog post Managing and Mitigating Security Vulnerabilities at Auth0.

Those who are familiar with JWT security know of the "none algorithm attack". Auth0 apparently blocklisted "alg":"none", but did that in a case sensitive manner. Setting "alg":"nonE" in the JWT header completely bypassed the signature check, which Insomnia Security (now CyberCX) found out in 2019. See their blog post JSON Web Token Validation Bypass in Auth0 Authentication API (https://cybercx.co.nz/blog/json-web-token-validation-bypass-in-auth0-authentication-api/) and Auth0's response in their blog post Insomnia Security Disclosure.

Auth0 also had a "algorithm confusion" vulnerability when validating JWT signatures. See their security bulletin CVE-2022-23539, CVE-2022-23541, CVE-2022-23540: Security Update for jsonwebtoken or the guest blog post by Tim McLean who found the problem: Critical vulnerabilities in JSON Web Token libraries.

There have been several other JWT validation vulnerabilities in Auth0 libraries in the past:

- CVE-2020-15084: Security Update for express-jwt Library
- CVE-2019-7644: Security Vulnerability in Auth0-WCF-Service-JWT
- CVE-2019-13483: Security Vulnerability in Passport-SharePoint
- CVE-2020-15240: Security Update for omniauth-auth0 JWT Validation

And one SAML (Security Assertion Markup Language) validation vulnerability as well:

- CVE-2022-23505: Security Update for passport-wsfed-saml2 Library

#### Timeline

All times are given in Central European Summer Time (CEST, UTC+02:00).

| Date | Event |
| --- | --- |
| 2023-JUL-20 (Day -2) | Found this authentication bypass vulnerability while preparing a report to Auth0 about the session fixation vulnerability |
| 2023-JUL-22 (Day 0) | Sent the vulnerability report to security@auth0.com, GPG encrypted to Auth0's new PGP key (from their security.txt file) |
| 2023-JUL-24 (Day 2) | Auth0 acknowledged receipt of the report: "Hi Laban! We are looking into the report and will get back to you soon. Thanks for your efforts in keeping Auth0 secure!" |
| 2023-AUG-05 (Day 14) | Okta (owners of Auth0) publish their blog post Saying "No Thanks" to nOAuth as a response to that Microsoft-specific vulnerability which is similar to this one. |
| 2023-AUG-28 (Day 37) | I reminded Auth0: "Now I'm back after my vacation. Did you manage to reproduce the authentication bypass vulnerability? Do you have any questions for me? Feel free to use my PGP key if you want to discuss anything sensitive." |
| 2023-OCT-10 (Day 80) | Another reminder to Auth0, this time with particular email recipients included who I corresponded with successfully in the session fixation matter. "I saw that you've changed the behavior of the Auth0 service so that it is no longer vulnerable to the authentication bypass problem I found. Do you agree that you have successfully mitigated it, or is it only a partial fix? Can you please comment on the vulnerability and tell me if I got some assumptions wrong? The disclosure deadline is in ten days (20th of October). If you need more time to implement a complete fix, please tell me." |
| 2023-OCT-18 (Day 88) | Auth0 asked me about this vulnerability when doing some follow-up on the session fixation problem. Note that this was the first time in 86 days that they got back to me regarding authentication bypass. Not so "soon" according to me. "Is there a second article you're working on as well? What is the timeline for that?" |
| 2023-OCT-20 (Day 90) | 90 days had passed since sending the vulnerability report to Auth0 |
| 2023-OCT-24 (Day 94) | Publication of this blog post |
