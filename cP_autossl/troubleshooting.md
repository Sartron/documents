# Troubleshooting
AutoSSL doesn't really act up, but when it does it can be difficult to sift through the logs and determine what the issue could be. This section is dedicated to providing guidance on how to read through the logs and find the source of any troubles.

## Investigation
AutoSSL logs are stored in `/var/cpanel/logs/autossl`. Alternatively, you can access _WHM » SSL/TLS » Manage AutoSSL » Logs_ and read the logs for an AutoSSL check there.  
Most AutoSSL errors are a result of bad DNS or a bad webserver setup. The former can be resulting from nonexistent records on some domain names, and the latter is most commonly due to interfering redirects, 403 blocks, or IP denials. Wordpress "security plugins" is a common source of AutoSSL issues.

## Error Dump
This is an error dump with server-specific details removed. When reading these, you will see blank areas which is where data such as a username, domain, IP, etc. where usually be present.  
For errors that state 'because of an error', the number **000** is supposed to reference an HTTP status code, and then the parantheses will have the name of the status code.

### DNS Errors
These types of errors can generally only need to be investigated using only `dig` or `nslookup`.
* The domain does not have a valid A record.
```
The domain “” failed domain control validation: “” does not resolve to any IPv4 addresses on the internet.
```
* The domain's A record does not point to the server.
```
The system failed to fetch the DCV (Domain Control Validation) file at “” because of an error: [_2]. The domain “” resolved to an IP address “” that does not exist on this server.
The system failed to fetch the DCV (Domain Control Validation) file at “”, which was redirected from “”, because of an error: [_3]. The domain “” resolved to an IP address “” that does not exist on this server.
The system queried for a temporary file at “”, but the web server responded with the following error: 000 (). A DNS (Domain Name System) or web server misconfiguration may exist. The domain “” resolved to an IP address “” that does not exist on this server.
The system queried for a temporary file at “”, which was redirected from “”. The web server responded with the following error: 000 (). A DNS (Domain Name System) or web server misconfiguration may exist. The domain “” resolved to an IP address “” that does not exist on this server.
```

### DCV Errors
These types of errors usually require reviewing webserver configuration and logs for more assistance. Apache domlogs can be a great help.
* Generic DCV error.
```
The system failed to fetch the DCV (Domain Control Validation) file at “” because of an error: [_2].
The system failed to fetch the DCV (Domain Control Validation) file at “”, which was redirected from “”, because of an error: [_3].
The system queried for a temporary file at “”, but the web server responded with the following error: 000 (). A DNS (Domain Name System) or web server misconfiguration may exist.
The system queried for a temporary file at “”, which was redirected from “”. The web server responded with the following error: 000 (). A DNS (Domain Name System) or web server misconfiguration may exist.
```

### cPanel-Based Errors
* Apache template for the domain does not exist within `/var/cpanel/userdata`.
```
load_userdata() failed - check /usr/local/cpanel/logs/error_log for additional details!
```
* User is suspended.
```
“” is suspended.
```
* '_Allow AutoSSL to replace invalid or expiring non-AutoSSL certificates_' is not enabled.
```
AutoSSL will ignore this website because this server’s AutoSSL did not provide the certificate for it.
```

### Other Errors
* Domain name was excluded from AutoSSL by the user.
```
The certificate for the website “” will not contain the domain “” because the current configuration excludes this domain.
```

### Unparsed Errors
These are unparsed, raw errors.
```
“[_1]” has [quant,_2,domain,domains], which exceeds the maximum number ([_3]) of domains per certificate for the “[_4]” AutoSSL provider. This system will include those [quant,_3,domain,domains] on the certificate that appear to be the website’s most important. To allow AutoSSL to secure each domain, divide the [quant,_2,domain,domains] among separate websites. (The websites can all serve the same content from the same document root.)
Checking websites for “[_1]” …
The system has completed the AutoSSL check for “[_1]”.
“[_1]” does not have the required [numerate,_2,feature,features] [list_and_quoted,_3].
“[_1]” is suspended.
The system failed to load the “[_1]” account’s data file because of an error: [_2]
The [quant,_1,website,websites] owned by “[_2]” [numerate,_1,has a valid,have valid] SSL [numerate,_1,certificate,certificates].
The website owned by “[_1]” has a valid SSL certificate.
AutoSSL cannot add any new domains to SSL coverage for the website “[_1]”.
All of “[_1]”’s unsecured domains failed domain control validation. AutoSSL will skip this website.
The current SSL certificate for “[_1]” secures the domain “[_2]”. However, this domain failed local domain control validation. In order to maintain SSL domain coverage for this domain, the system will not attempt to replace the current certificate.
“[_1]” has [quant,_2,domain,domains]. The “[_3]” AutoSSL provider can only create certificates with up to [quant,_4,domain,domains]. Because “[_3]” cannot secure all [numf,_5] currently secured non-wildcard [numerate,_5,domain,domains], and this website’s current certificate is valid, AutoSSL will skip this website.
AutoSSL will defer the renewal of “[_1]”’s certificate because [quant,_2,domain,domains] ([list_and,_3]) that the current certificate secures failed DCV. If AutoSSL renewed the certificate now, [numerate,_2,that domain,those domains] would lose SSL coverage. AutoSSL will defer “[_1]”’s certificate renewal until [datetime,_4,datetime_format_short] UTC ([quant,_5,day,days] before expiry) or until all of “[_1]”’s currently secured domains pass DCV.
This website’s SSL certificate lacks the following [numerate,_1,domain,domains]: [join,~, ,_2]. AutoSSL will not replace a certificate that an installed AutoSSL provider did not generate.
This website’s SSL certificate lacks the following [numerate,_1,domain,domains]: [join,~, ,_2]. AutoSSL will not replace a certificate that an installed AutoSSL provider did not generate unless it expires within [quant,_3,day,days].
The website “[_2]”, owned by “[_1]”, has a valid SSL certificate, but additional SSL coverage may be possible for the [numerate,_3,domain,domains] [list_and_quoted,_4].
AutoSSL cannot provide SSL certificates for wildcard domains, so AutoSSL will skip this website.
The system will attempt to replace this certificate with one that includes the additional non-wildcard [numerate,_1,domain,domains], but AutoSSL cannot provide SSL certificates for the wildcard [numerate,_2,domain,domains].
The website “[_2]”, owned by “[_1]”, has a faulty SSL certificate ([_3]).
AutoSSL cannot provide wildcard SSL certificates, and “[_1]” has no non-wildcard domain names, so AutoSSL will skip this website.
The certificate file for the website “[_2]”, owned by “[_1]”, is empty. This may indicate filesystem corruption. The file’s location is “[_3]”.
The certificate file for the website “[_2]”, owned by “[_1]”, does not exist at the expected path, “[_3]”.
The website “[_2]”, owned by “[_1]”, has no SSL certificate.
The domain “[_1]” failed domain control validation: [_2]
The validation required [numf,_1] HTTP [numerate,_1,redirect,redirects], but the AutoSSL provider “[_2]” does not permit HTTP redirects.
When the system accessed the “[_1]” URL, it redirected to the “[_2]” URL.
The validation required [numf,_1] HTTP [numerate,_1,redirect,redirects], but the AutoSSL provider “[_2]” only permits [quant,_3,redirect,redirects].
This system has AutoSSL disabled.
This system has AutoSSL set to use “[_1]”.
The cPanel user named “[_1]” does not exist on this system.
```