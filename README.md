# api-secure
Build APIs with Security

First of all, this project is to secure deeply for apis
## CORS - CROSS-ORIGIN RESOURCE SHARING
CORS is a set of browser controls set by web servers
CORS defines what responses are allowed from where
- Origins
- Credentials
- Methods
- Headers

This is important part to protect info from API or web server and prevent Cross-site Scripting (XSS) Attack
- User data
- Intellectual property
- Branding

We should use CORS to helps protect from 
- **Malicious System** (*API*)
- **Malicious Site** (*CORS Blocked*)

What things we need to do is set headers, define what is allowed such as:
- Access-Control-Allow-Origin: *, test.com
- Access-Control-Allow-Headers: **x-authentication-token**, **Authorization**, ...
- Access-Control-Allow-Methods: **GET**, **POST**, **DELETE**

## Error Disclosure
Error Disclosure = revealing technology and code in an error
- Erroring out is necessary
- Intentional error handling is good
- Developers and Customers are separate error message audiences

For example:
### Python 
```
try:
    print(x)
except E:
    print("An exception occurred", E)
```
### Nodejs
```
try{
    fs.readFile('sample.txt',
        function(err, data){
            if (err) throw err;
        });
} catch (err) {
    console.log("In Catch Block")
    console.log(err)
}
```
### Go
```
res, err := getContentFromFileName("sample.txt")
if err != nil {
    fmt.Println("Error occurred ", err.Error())
}
```

We should do it because raw error was returned FE, it helps attacker catch vulnerability in our server and find out a way to be used maliciously.
There are some ways to do it easily
- API: Provide generic error message
- Github: Show *404 Not Found* Page if we don't have access to that page

**Best Practices**
- Error early
- Include useful information for developers in an error
- Log the error
- Display something specifically crafted for customers on error

## Server Information Leak
Server Information Leak = tech stack advertisement in headers
- Web servers advertise in headers by default
- Appliances can also add headers
- These headers have no benefits
- Detecting and removing them is easy

Sometimes, it's assumed that server's returning server header like uvicorn - nginx - x-powered-by stuff like that
It is dangerous for us when attackers know our framework, version, and cve[^1] vulnerabilities in this framework.


We resolve this issue by remove `server-header` from our application

## Insecure Cookies
Insecure Cookies = Cookies stored without restrictive security settings
- Secure Cookies
- HTTP Only
- SameSite, Domains, Subdomains

**COOKIES DECODING**
this includes many interest things 
- Key/Fields
- Sort data types
    - Unique ID String
    - Numeric ID 
    - Boolean 
    - Encoded Data
- Decoding data 
    - Delimiters
    - Base64 decode and similar
Some problems occurs
1. **Cookie forgin/Fuzzing** (Trusting cookie data)
2. **Data harvesting, different site** (httponly + no domains)
3. **Data harvesting, through XSS** (httponly)
4. **Data harvesting in transite** (Secure flag)

**SOLUTIONS**
1. Treat cookies as untrusted user data
2. Be restrictive on waht data is stored in cookies
3. Analyze your cookies from an offensive mindset
Must be using SSL/TLS to protect cookies better

## Path Traversal
Path Traversal = allowing access ot unintended paths on a server
- Directory traversal
- Direct file reference

- Server configuration
- Defensive coding
- Spec
```
# Malicious Cookie
GET /vulnerability HTTP/1.0
Cookie:
TEMPLATE="../../../etc/passwd
```
Append malicious path 

**PROBLEM**
1. Input data validation
2. Server config
    > Directory listings, and allowed include paths
3. Dynamic paths in source code
4. Loose Spec "String" allows attacks

**SOLUTION**
1. Scrub input data
    - Specify what allowed. Enum/Array/List
    - Don't try filtering what's bad
2. Disable server directory listings
3. Don't store anything sensitive at web root
4. Put the web root on a separate drive than system
5. Remove variables from paths in source code
6. When in doubt, error

## Rate Limiting
Rate limiting = Setting inbound limitations to ensure a service level and reliability
- RateLimit headers
    - Ratelimit-Limit: 10
    - Ratelimit-Remaining: 2
    - Ratelimit-Reset: 7
- HTTP Status = 429
- Scope: user, origin, global, resource, endpoint
- Server Side Throttling

We should use Rate Limit for
- Service availability
- Security 
- Budge
- Reduce Resource Consumption (**OWASP-API-4**)

Limiting in Layers
1. Gateway / Network Appliance
2. Service
3. Server
4. Endpoint
5. User
6. Client Signature
7. Quota
8. Logic

Some scenarios can be occurred in our server if we don't implement Rate Limit
1. Application DDoS
2. Impatient User
3. Brute Force
4. Budget Server

> [!TIP] **TECH**
> - Avoid SQL Operations to manage throttling
> - Avoid disk operations
> - Include IP With User
> - Use cache where appropriate (same query) - TTL


## Reference 
[^1]: [cve details](https://www.cvedetails.com/)