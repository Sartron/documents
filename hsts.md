# HTTP Strict Transport Security

## Overview
HSTS is a web mechanism that allows the the webserver to declare that the site should only be viewed over HTTPS. This works in the following manner:
1. Client views site over HTTPS.
2. Webserver sends Strict-Transport-Security headers to client.
3. Client redirects further requests to site over HTTPS for as long as the max-age lasts.
This is **not** the same thing as a force-HTTPS redirect. This redirect is occurring at the browser-level rather than at the server-level.

## Requirements
To set up HSTS on a site, the webserver needs to have some type of method to send headers to the client.
| Webserver | Module                  | Command    | Module Check              |
|-----------|-------------------------|------------|---------------------------|
| Apache    | mod_headers             | Header     | httpd -M | grep 'headers' |
| nginx     | ngx_http_headers_module | add_header |                           |
Additionally, the web browser being used must support HSTS as well. All modern web browsers support HSTS.

## Setup and Example
_Note: This section will only be covering setting up HSTS for Apache._  

Provided the webserver meets the requirements for sending HSTS, you will need to add the relevant command to wherever the webserver is loading the site from.  
For Apache, this can be from a .htaccess file, virtual host include, or the actual virtual host for the site. In most scenarios, you will be setting this up within a .htaccess for a single domain.
```
Header set Strict-Transport-Security "max-age=31536000; includeSubDomains"
```
The above code is entirely sufficient in setting up HSTS and is all that is needed. For an explanation of the above:
* `Header set Strict-Transport-Security`: Send header Strict-Transport-Security to the client.
* `"max-age=31536000`: The amount in seconds that the client will recognize the host as an HSTS host. 31536000 seconds is 1 year.
* `; includeSubDomains"`: Semi-colon allows an additional directive to be appended. The directive includeSubDomains causes all subdomains (e.g. *.domain.tld) under the HSTS host to be recognized as an HSTS host as well.

## Verification
How do you verify HSTS is working? There are multiple ways:
* See if HTTP redirects to HTTPS after visiting HTTPS once.
* Check your browser's HSTS storage to see if the host is there.
 * Firefox: `%appdata%\Mozilla\Firefox\Profiles\<profile>\SiteSecurityServiceState.txt`
 * Chrome: `chrome://net-internals/#hsts`
* Use curl or browser developer console to see if the header is being received.
 * curl
 ```
 [user@server ~]# curl -sI https://domain.tld | grep 'Strict-Transport-Security'
 Strict-Transport-Security: max-age=31536000; includeSubDomains
 ```
* [SSL Labs](https://www.ssllabs.com/) can check if HSTS is enabled on the provided hostname.

## Citations
The following links may be referenced for further information on this topic:
* https://tools.ietf.org/html/rfc6797#page-15
* https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security
* https://raymii.org/s/tutorials/HTTP_Strict_Transport_Security_for_Apache_NGINX_and_Lighttpd.html