---
title: "Ultimate 401 and 403 bypass methods"
author: Klaudia Kloc (Vidoc Security Lab)
url: https://blog.vidocsecurity.com/blog/401-and-403-bypass-how-to-do-it-right
date_published: 2023-05-09
date_scraped: 2026-11-17
platform: other
disclosed_report: none
bounty: not stated
vuln_class: access-control
---

<!-- Raw scraped text below. Public, non-paywalled content only. -->
<!-- Scrape note: full article publicly readable. Payload lists in the original are truncated with "See more in the module" (Vidoc tool). Promo banners and "more articles" omitted. -->

As a security researcher, I absolutely love the rush of discovering a suspicious endpoint during reconnaissance (which is super easy with VIDOC tool ;). It's exciting to think that you might have stumbled upon something important. However, that excitement can quickly turn into frustration when you're met with a 401 or 403 HTTP response code. Trust me, I've been there.

But, over time, I've learned that there are ways to work around these error codes. I want to share some practical tips and techniques that I learned doing research. So, let's get started!

> If you know what HTTP status codes are and what the 401 or 403 response means, skip the next section and go straight to the bypass techniques.

## Understanding 401 and 403 HTTP Response Codes

The 401 Unauthorized and 403 Forbidden HTTP response codes are both related to access control, but they have distinct meanings and implications.

The 401 Unauthorized response code indicates that the client making the request is not authenticated and therefore does not have the necessary credentials to access the requested resource. This response is typically used when a server requires authentication to access a resource, and the client has not provided valid credentials or has not yet authenticated.

On the other hand, the 403 Forbidden response code indicates that the client is authenticated, but does not have sufficient permissions to access the requested resource. This response is typically used when a server wants to prevent a client from accessing a resource, either because the client does not have the required permissions or because the resource is being intentionally hidden or restricted.

### What does it mean to bypass 401 or 403?

It means to get access to the resource without explicit authentication or authorization. There are several methods that can be used to bypass this error.

## Common methods of bypass 401&403 status codes

### HTTP Methods/Verbs fuzzing.

One of the best known is **HTTP Methods/Verbs fuzzing**. If you send a request with `GET` method and response is 401 and 403 you may want to send it again using a different method like `POST`, `PUT` , `PATCH` and see what happens.

**Request**:

```
GET /admin HTTP/1.1
Host: example.com
...
```

**Response**:

```
HTTP/1.1 403 Forbidden
Server: Apache/2.4.41
Content-Length: 9
Connection: close

Access Denied
```

### You can simply change method to POST

**Request**:

```
POST /admin HTTP/1.1
Host: example.com
...
```

**Response**:

```
HTTP/1.1 200 OK
Server: Apache/2.4.41
Content-Length: 564
Connection: close

<!DOCTYPE html>
<html>
<head>
    <title>Admin dashboard</title>
...
```

### Other HTTP methods to check:

```
GET
HEAD
POST
PUT
DELETE
CONNECT
OPTIONS
TRACE
PATCH
FOO # non existant method also might work
```

### User-Agent fuzzing

Next to HTTP Methods/Verbs fuzzing another useful method could be **User-Agent fuzzing**. Sometimes developers want to serve different content depending on what kind of browser/OS you use to access the web app. In case of misconfiguration you might be able to get access to the resource only by modifying User Agent.

```
Mozilla/5.0 (X11; Linux i686; U;rv: 1.7.13) Gecko/20070322
  Kazehakase/0.4.4.1

Mozilla/5.0 (X11; U; Linux 2.4.2-2 i586; en-US; m18) Gecko/20010131
  Netscape6/6.01

Mozilla/5.0 (X11; U; Linux i686; de-AT; rv:1.8.0.2) Gecko/20060309
  SeaMonkey/1.0

Mozilla/5.0 (X11; U; Linux i686; en-GB; rv:1.7.6) Gecko/20050405
  Epiphany/1.6.1 (Ubuntu) (Ubuntu package 1.0.2)

Mozilla/5.0 (X11; U; Linux i686; en-US; Nautilus/1.0Final)
  Gecko/20020408

Mozilla/5.0 (X11; U; Linux i686; en-US; rv:0.9.3) Gecko/20010801

See more in the module below...
```

### HTTP Headers fuzzing

HTTP Headers fuzzing is another popular method, this technique involves sending a request to the target server with a modified HTTP header. X-Forwarded-For, Referer and Authorization headers are the most popular one to be modified. X-Forwarded-For can be fuzzed to bypass security controls that rely on IP address filtering. By changing the IP address in this header, an attacker can attempt to bypass any security controls that are based on IP address filtering. Example payloads:

```
Headers:
- X-Forwarded-For
- X-Forward-For
- X-Forwarded-Host
- X-Forwarded-Proto
- Forwarded
- Via
- X-Real-IP
- X-Remote-IP
- X-Remote-Addr
- X-Trusted-IP
- X-Requested-By
- X-Requested-For
- X-Forwarded-Server

Values:
- 10.0.0.0
- 10.0.0.1
- 127.0.0.1
- 127.0.0.1:443
- 127.0.0.1:80
- localhost
- 172.16.0.0

See more in the module below...
```

### Path Fuzzing and creative string literals.

This is another useful method, you can try to bypass 401 or 403 status code by adding symbols to find alternative paths. example.com/admin gives you 403? Try adding `/%2e/` or `/%252e/` to the path: `example.com/%252e/`admin and check results. You can also try other creative literals.

**Example paths to fuzz**

```
/../
/...
/..%00
/..%01
/..%0a
/..%0d
/..%09
/~root
/~admin
/%20/
/%2e%2e/
/%252e%252e/
/%c0%af/
/%e0%80%af

See more in the module below...
```

**Why this works?**

The "/%2e/" or "/%252e/" sequence represents the URL-encoded version of the "." character, which is used to represent the current directory in file paths. By using this sequence in the URL path, the server may interpret the request as accessing a different path than the one intended, potentially bypassing access controls that are based on the original path.

### Downgrade the protocol version.

HTTP/2 won't let you in? Try 1.1. You can also try all other techniques along with downgrading the protocol, one useful one might be to downgrade to 1.0 and remove the Host header at all - it can cause unexpected behaviour of the server. This technique was described in detail in this research article (https://infosecwriteups.com/403-bypass-lyncdiscover-microsoft-com-db2778458c33).

```
0.9
1.0
1.1
2
```

## Less known 401 and 403 bypass techniques

### Case switching technique

It involves altering the case of characters in the URL path to try and bypass access controls. `example.com/user` gives you 401? Try `example.com/User`, `example.com/%75ser` or `example.com/USer` etc.

### HTTP request smuggling

HTTP request smuggling can be used to manipulate the behaviour of web servers and web application firewalls. It involves sending malformed HTTP requests that exploit discrepancies in the way that different components in the HTTP request processing chain handle request data. We won't get deep into that one, there are several techniques that can be used to exploit that issue and it deserves a separate article. What you need to know is that in the case of 401 and 403 status codes, you could use HTTP request smuggling to bypass authentication and authorization checks. Check out this research (https://medium.com/@siratsami71/the-easiest-way-i-used-to-bypass-an-admin-panel-93d4297ed4a6).

### Exploiting Hop-by-Hop request header.

To understand this technique one should first get familiar with hop-to-hop headers. Hop-by-hop headers are HTTP headers that are intended for communication between a client and a server, and that are not forwarded by intermediaries such as proxies or caches. These headers are called "hop-by-hop" because they are processed on a per-hop basis, meaning that each intermediary along the path of the request will see and potentially modify the headers as they pass through. The HTTP/1.1 specification defines the following headers as hop-by-hop headers:

> Connection, Keep-Alive, Proxy-Authenticate, Proxy-Authorization, TE, Trailer, Transfer-Encoding

When a proxy receives a request containing one of these headers, it should process it and then remove the header before forwarding the request to the next server in the chain.

Note

The **HTTP/2 protocol** has a different mechanism for handling headers, with the concept of "pseudo-headers" that are treated differently from regular headers.

What's interesting, you can define what headers should be removed by adding them to Connection header, like this: [image: Connection header listing a header to strip]

Using this trick, you may be able to bypass 401 and 403 status codes, as long as some server in the request chain relies on the header that the proxy adds/intermediary server adds, but you define it to be removed. However, it is important to note that just because you can remove the header somewhere in the chain does not necessarily mean the host is vulnerable. This is only the first step in exploiting the underlying issue, and it is necessary to explore further to determine if it can be used to bypass the authorization check. Additionally, it is important to recognize that this technique may cause unexpected issues on the server-side, depending on how the server handles the removed header. If you want to know more about this method Nathan Davison did excellent research (https://nathandavison.com/blog/abusing-http-hop-by-hop-request-headers) on that.

Example payloads to be fuzzed with Connection header:

```
Hop by hop headers:
- Accept
- Accept-Application
- Accept-Charset
- Accepted
- Accept-Encoding
- Accept-Encodxng
- Accept-Language
- Accept-Ranges
- Accept-Version
- Access-Control-Allow-Credentials
- Access-Control-Allow-Headers
- Access-Control-Allow-Methods
- Access-Control-Allow-Origin
- Access-Control-Expose-Headers
```

### Spring Framework specific bypass technique

Versions < 5.3 have `useSuffixPatternMatch` setting set to true by deafault. It means that methods mapped for example to `/admin` would also match `/admin[.].*`. It can be used to bypass access restrictions, source and more details about this technique can be found here (https://raw.githubusercontent.com/Mike-n1/tips/main/SpringAuthBypass.png).

## Any other ideas?

In conclusion, bypassing 401 and 403 status codes can be a challenging task, but there are many methods that can be employed to achieve it. We covered the most common techniques, included some of them in our tool. We want to create the ultimate guide and gather all known techniques in one place, so if you have any other ideas - let us know!
