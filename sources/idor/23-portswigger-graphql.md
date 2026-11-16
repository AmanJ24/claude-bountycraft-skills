---
title: GraphQL API vulnerabilities | Web Security Academy
author: PortSwigger
url: https://portswigger.net/web-security/graphql
date_published: not stated
date_scraped: 2026-11-16
platform: portswigger
disclosed_report: none
bounty: not applicable
vuln_class: idor
---

<!-- Raw scraped text below. Public, non-paywalled content only.
     This file is source material, never copied into skills verbatim. -->

# GraphQL API vulnerabilities

GraphQL vulnerabilities generally arise due to implementation and design flaws. For example, the introspection feature may be left active, enabling attackers to query the API in order to glean information about its schema.

GraphQL attacks usually take the form of malicious requests that can enable an attacker to obtain data or perform unauthorized actions. These attacks can have a severe impact, especially if the user is able to gain admin privileges by manipulating queries or executing a CSRF exploit. Vulnerable GraphQL APIs can also lead to information disclosure issues.

## Finding GraphQL endpoints

As GraphQL APIs use the same endpoint for all requests, finding the endpoint is valuable.

### Universal queries

If you send `query{__typename}` to any GraphQL endpoint, it will include the string `{"data": {"__typename": "query"}}` somewhere in its response. This is known as a universal query. The query works because every GraphQL endpoint has a reserved field called `__typename` that returns the queried object's type as a string.

### Common endpoint names

- `/graphql`
- `/api`
- `/api/graphql`
- `/graphql/api`
- `/graphql/graphql`

If these don't return a GraphQL response, try appending `/v1` to the path. GraphQL services will often respond to any non-GraphQL request with a "query not present" or similar error.

### Request methods

Best practice is for production GraphQL endpoints to only accept POST requests with content-type `application/json` (protects against CSRF). Some endpoints may accept GET requests or POST requests that use `x-www-form-urlencoded`. If you can't find the endpoint with POST, try the universal query using alternative HTTP methods.

### Initial testing

Explore the web interface in Burp's browser and use the HTTP history to examine the queries that are sent.

## Exploiting unsanitized arguments

If the API uses arguments to access objects directly, it may be vulnerable to access control vulnerabilities. A user could potentially access information they should not have simply by supplying an argument that corresponds to that information. This is sometimes known as an insecure direct object reference (IDOR).

Example product query:

```
query {
    products {
        id
        name
        listed
    }
}
```

The product list returned contains only listed products.

```
{
    "data": {
        "products": [
            { "id": 1, "name": "Product 1", "listed": true },
            { "id": 2, "name": "Product 2", "listed": true },
            { "id": 4, "name": "Product 4", "listed": true }
        ]
    }
}
```

From this information, we can infer: products are assigned a sequential ID; product ID 3 is missing from the list, possibly because it has been delisted.

By querying the ID of the missing product, we can get its details, even though it is not listed on the shop and was not returned by the original product query.

```
query {
    product(id: 3) {
        id
        name
        listed
    }
}
```

```
{
    "data": {
        "product": {
            "id": 3,
            "name": "Product 3",
            "listed": no
        }
    }
}
```

## Discovering schema information

The best way to piece together the underlying schema is to use introspection queries. Introspection is a built-in GraphQL function that enables you to query a server for information about the schema. It can also disclose potentially sensitive data, such as description fields.

### Using introspection

Query the `__schema` field, available on the root type of all queries. Burp can generate introspection queries for you.

### Probing for introspection

```
{
    "query": "{__schema{queryType{name}}}"
}
```

If introspection is enabled, the response returns the names of all available queries.

### Running a full introspection query

```
query IntrospectionQuery {
    __schema {
        queryType {
            name
        }
        mutationType {
            name
        }
        subscriptionType {
            name
        }
        types {
         ...FullType
        }
        directives {
            name
            description
            args {
                ...InputValue
        }
        onOperation  #Often needs to be deleted to run query
        onFragment   #Often needs to be deleted to run query
        onField      #Often needs to be deleted to run query
        }
    }
}

fragment FullType on __Type {
    kind
    name
    description
    fields(includeDeprecated: true) {
        name
        description
        args {
            ...InputValue
        }
        type {
            ...TypeRef
        }
        isDeprecated
        deprecationReason
    }
    inputFields {
        ...InputValue
    }
    interfaces {
        ...TypeRef
    }
    enumValues(includeDeprecated: true) {
        name
        description
        isDeprecated
        deprecationReason
    }
    possibleTypes {
        ...TypeRef
    }
}

fragment InputValue on __InputValue {
    name
    description
    type {
        ...TypeRef
    }
    defaultValue
}

fragment TypeRef on __Type {
    kind
    name
    ofType {
        kind
        name
        ofType {
            kind
            name
            ofType {
                kind
                name
            }
        }
    }
}
```

If introspection is enabled but the above query doesn't run, try removing the `onOperation`, `onFragment`, and `onField` directives. Many endpoints do not accept these directives as part of an introspection query.

### Visualizing introspection results

A GraphQL visualizer (http://nathanrandal.com/graphql-visualizer/) takes introspection results and produces a visual representation of relationships between operations and types.

### Suggestions

Even if introspection is entirely disabled, you can sometimes use suggestions to glean information on an API's structure. Suggestions are a feature of the Apollo GraphQL platform in which the server can suggest query amendments in error messages (for example, `There is no entry for 'productInfo'. Did you mean 'productInformation' instead?`). Clairvoyance (https://github.com/nikitastupin/clairvoyance) uses suggestions to automatically recover all or part of a GraphQL schema, even when introspection is disabled. In Apollo Server v4 and above, suggestions can be disabled using the `hideSchemaDetailsFromClientErrors` option.

LABS: Accessing private GraphQL posts (Apprentice); Accidental exposure of private GraphQL fields (Practitioner)

## Bypassing GraphQL introspection defenses

If you cannot get introspection queries to run, try inserting a special character after the `__schema` keyword. When developers disable introspection, they could use a regex to exclude the `__schema` keyword. Try characters like spaces, new lines and commas, as they are ignored by GraphQL but not by flawed regex. If the developer has only excluded `__schema{`, the below introspection query would not be excluded.

```
{
    "query": "query{__schema
    {queryType{name}}}"
}
```

If this doesn't work, try running the probe over an alternative request method, as introspection may only be disabled over POST. Try a GET request, or a POST request with content-type `x-www-form-urlencoded`.

```
GET /graphql?query=query%7B__schema%0A%7BqueryType%7Bname%7D%7D%7D
```

LAB: Finding a hidden GraphQL endpoint (Practitioner)

## Bypassing rate limiting using aliases

Aliases enable you to return multiple instances of the same type of object in one request. Aliases can be used to brute force a GraphQL endpoint. Some rate limiters work based on the number of HTTP requests received rather than the number of operations performed. Because aliases effectively enable you to send multiple queries in a single HTTP message, they can bypass this restriction.

```
query isValidDiscount($code: Int) {
    isValidDiscount(code:$code){
        valid
    }
    isValidDiscount2:isValidDiscount(code:$code){
        valid
    }
    isValidDiscount3:isValidDiscount(code:$code){
        valid
    }
}
```

LAB: Bypassing GraphQL brute force protections (Practitioner)

## GraphQL CSRF

GraphQL can be used as a vector for CSRF attacks, whereby an attacker creates an exploit that causes a victim's browser to send a malicious query as the victim user.

### How do CSRF over GraphQL vulnerabilities arise?

CSRF vulnerabilities can arise where a GraphQL endpoint does not validate the content type of the requests sent to it and no CSRF tokens are implemented. POST requests with content type `application/json` are secure against forgery as long as the content type is validated. However, alternative methods such as GET, or any request with content type `x-www-form-urlencoded`, can be sent by a browser and so may leave users vulnerable if the endpoint accepts these requests.

LAB: Performing CSRF exploits over GraphQL (Practitioner)

## Preventing GraphQL attacks

- If your API is not intended for use by the general public, disable introspection.
- If your API is public, review the schema to make sure it does not expose unintended fields.
- Make sure that suggestions are disabled.
- Make sure that your API's schema does not expose any private user fields, such as email addresses or user IDs.

### Preventing GraphQL brute force attacks

- Limit the query depth.
- Configure operation limits (maximum number of unique fields, aliases, and root fields).
- Configure the maximum amount of bytes a query can contain.
- Consider implementing cost analysis.

### Preventing CSRF over GraphQL

- Your API only accepts queries over JSON-encoded POST.
- The API validates that content provided matches the supplied content type.
- The API has a secure CSRF token mechanism.
