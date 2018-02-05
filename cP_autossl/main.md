# Introduction
AutoSSL is a feature introduced in cPanel 58 which automatically installs domain-validated SSL certificates on users' domains for the Apache, Dovecot, and Exim services.
This article will be written assuming that the server is configured to use cPanel as its certificate provider and will only provide limited information for third-party providers such as Let's Encrypt.

# Requirements
In order to use AutoSSL, you must be the server environment must be using at least cPanel 58. That means the following conditions should be met:
* RHEL/CentOS/CloudLinux 6+ (64-bit)
* Support for SNI  

If the server you are accessing does not meet any of the above requirements, it cannot use AutoSSL.

# Usage
AutoSSL may be manipulated through both GUI and CLI, although both interface types provide access to the same functionality and information.
Most tasks concerning AutoSSL should ideally be performed through the GUI for simplicity sake, although there are equivalent CLI commands for most functions.

## GUI
Only root is able to manage AutoSSL _from WHM_. This is accessible through **WebHost Manager » SSL/TLS » Manage AutoSSL**.  
Normal cPanel users have limited AutoSSL functionality within **cPanel » Security » SSL/TLS Status**.

### WHM: Providers
This section will provide the available AutoSSL providers installed on the server. The following is a list of providers you may see:
* cPanel: Natively supported as of cPanel 58.
* Let's Encrypt: Supported as an official plugin as of cPanel 58.0.17.

cPanel has developed functionality to allow for third-party AutoSSL providers. This is documented in their article Guide to Third-Party AutoSSL Provider Modules.
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/whm_providers.png)

### WHM: Options
This section will provide the available options for use with AutoSSL provider. You must be using **at least cPanel 60** in order to view this menu. Currently, the following options are available:
* _Allow AutoSSL to replace invalid or expiring non-AutoSSL certificates_: Allows AutoSSL to replace certificates it did not issue.

![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/whm_options.png)

### WHM: Logs
Contains logs of AutoSSL's queue runs if there are any available. This is the primary troubleshooting tool for investigating potential problems with AutoSSL.
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/whm_logs.png)

### WHM: Manage Users
This section may be used to manage the availability of AutoSSL to users on the server. The following settings are available for determining whether or not AutoSSL is enabled for the user:
* _Enable AutoSSL_: Force AutoSSL to be enabled for the user.
* _Disable AutoSSL_: Force AutoSSL to be disabled for the user.
* _Reset to Feature List Setting_: Enable or disable AutoSSL according to the user's set feature list. This is the default option.

Additionally, you can freely execute AutoSSL checks from here on any number of users. This is so that you do not need to wait for the AutoSSL cron to execute.
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/whm_manage.png)

#### WHM: Pending Queue
This section allows you to view the current AutoSSL queue on the server. This tab will only become visible provided you are using at least cPanel 62 **and cPanel is set as the provider**.
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/whm_queue.png)

#### cPanel: SSL/TLS Status
As of [cPanel 66](https://documentation.cpanel.net/display/66Docs/66+Release+Notes#id-66ReleaseNotes-Domain-specificAutoSSLexclusion), users are able to exclude and manage AutoSSL from within cPanel, provided they have access to SSL/TLS within cPanel.
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/cP-ssltls.png)

## CLI
Manipulating AutoSSL through the CLI is done largely through cPanel WHM API/UAPI calls, however some of these calls are just wrappers for backend programs and files.
This section will be covering the available API calls, as well as how to use and view backend files that control AutoSSL.

### WHM API 1
All of the following WHM API 1 calls that will be reviewed will be using YAML formatting. whmapi1 also supports XML & JSON formatting which should be taken into consideration when executing these commands.
This section does not cover all AutoSSL WHM API calls. It will only cover the needed commands that are needed to fulfill their GUI counterparts.

#### Providers
* `whmapi1 get_autossl_check_schedule`: Outputs the schedule at which AutoSSL is run.
```
  cron:
    - 45
    - 4
    - "*"
    - "*"
    - "*"
  next_time: 0000-00-00T00:00:00Z
```
Available in cPanel 58+: [WHM API 1 Functions - get_autossl_check_schedule](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_check_schedule)
* `whmapi1 get_autossl_providers`: Provides list of AutoSSL providers.
```
payload:
  -
    display_name: cPanel (powered by Comodo)
    enabled: 0
    module_name: cPanel
  -
    display_name: Let’s Encrypt™
    enabled: 0
    module_name: LetsEncrypt
    x_terms_of_service: https://acme-v01.api.letsencrypt.org/terms
    x_terms_of_service_accepted: https://acme-v01.api.letsencrypt.org/term
```
Available in cPanel 58+: [WHM API 1 Functions - get_autossl_providers](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_providers)
* `whmapi1 set_autossl_provider`: Sets the current AutoSSL provider to the specified *module_name*.  
Available in cPanel 58+: [WHM API 1 Functions - set_autossl_provider](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+set_autossl_provider)
* `whmapi1 disable_autossl`: Disables AutoSSL.
Available in cPanel 58+: [WHM API 1 Functions - disable_autossl](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+disable_autossl)

#### Options
* `whmapi1 set_autossl_metadata`: Used to add metadata to `/var/cpanel/autossl.json`.
```
[root@server ~]# python -m json.tool /var/cpanel/autossl.json | awk '/metadata/,/}/'
    "metadata": {
        "clobber_externally_signed": "0"
    },
[root@server ~]# whmapi1 set_autossl_metadata metadata_json={\"clobber_externally_signed\":1}
---
metadata:
  command: set_autossl_metadata
  reason: OK
  result: 1
  version: 1
[root@server ~]# python -m json.tool /var/cpanel/autossl.json | awk '/metadata/,/}/'
    "metadata": {
        "clobber_externally_signed": "1"
    },
```
Available in cPanel 60+: [WHM API 1 Functions - set_autossl_metadata](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+set_autossl_metadata)

#### Logs
* `whmapi1 get_autossl_logs_catalog`: Output list of logs pertaining to AutoSSL queue runs. A log with the username "*" indicates that all users were checked in this log.
```
payload:
  -
    in_progress: 0
    provider: cPanel
    start_time: 0000-00-00T00:00:00Z
    username: user
  -
    in_progress: 0
    provider: cPanel
    start_time: 0000-00-00T00:00:00Z
    username: "*"
```
Available in cPanel 58+: [WHM API 1 Functions - get_autossl_logs_catalog](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_logs_catalog)
* `whmapi1 get_autossl_log`: When provided with the *start_time* of an AutoSSL log, it retrieves each entry of the provided log.
```
  payload:
    -
      contents: "This system has AutoSSL set to use “cPanel (powered by Comodo)”.\n"
      indent: 0
      partial: 0
      pid: 00000
      timestamp: 0000-00-00T00:00:00Z
      type: out
    -
      contents: "The system has completed the AutoSSL check for “username”.\n"
      indent: 0
      partial: 0
      pid: 00000
      timestamp: 0000-00-00T00:00:00Z
      type: out
```
Available in cPanel 58+: [WHM API 1 Functions - get_autossl_log](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_log)
* `whmapi1 get_autossl_problems_for_user`: Retrieves the recorded problems preventing AutoSSL from issuing to domains under this user.
```
  problems_by_domain:
    -
      domain: cpanel.domain.tld
      log: 0000-00-00T00:00:00Z
      problem: “cpanel.domain.tld” does not resolve to any IPv4 addresses on the internet.
      time: 0000-00-00T00:00:00Z
    -
      domain: webmail.domain.tld
      log: 0000-00-00T00:00:00Z
      problem: “webmail.domain.tld” does not resolve to any IPv4 addresses on the internet.
      time: 0000-00-00T00:00:00Z
    -
      domain: www.domain.tld
      log: 0000-00-00T00:00:00Z
      problem: “www.domain.tld” does not resolve to any IPv4 addresses on the internet.
      time: 0000-00-00T00:00:00Z
    -
      domain: domain.tld
      log: 0000-00-00T00:00:00Z
      problem: “domain.tld” does not resolve to any IPv4 addresses on the internet.
      time: 0000-00-00T00:00:00Z
    -
      domain: webdisk.domain.tld
      log: 0000-00-00T00:00:00Z
      problem: “webdisk.domain.tld” does not resolve to any IPv4 addresses on the internet.
      time: 0000-00-00T00:00:00Z
    -
      domain: mail.domain.tld
      log: 0000-00-00T00:00:00Z
      problem: “mail.domain.tld” does not resolve to any IPv4 addresses on the internet.
      time: 0000-00-00T00:00:00Z
```
Available in cPanel 68+ [WHM API 1 Functions - get_autossl_problems_for_user](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_problems_for_user)

#### Manage Users
* `whmapi1 start_autossl_check_for_all_users`: Performs an AutoSSL check for all cPanel users that have AutoSSL enabled. This command executes the check and returns the PID of the background process performing the check.  
Available in cPanel 58+: [WHM API 1 Functions - start_autossl_check_for_all_users](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+start_autossl_check_for_all_users)
* `whmapi1 start_autossl_check_for_one_user`: Performs an AutoSSL check for one cPanel user. User must have AutoSSL enabled to be processed in the queue. This command executes the check and returns the PID of the background process performing the check.  
Available in cPanel 58+: [WHM API 1 Functions - start_autossl_check_for_one_user](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+start_autossl_check_for_one_user)

#### Pending Queue
* `whmapi1 get_autossl_pending_queue`: Outputs list of domains currently being processed within the AutoSSL queue.
```
pending_certificates:
  -
    domain: domain.tld
    order_item_id: 000000000
    request_time: 0000-00-00T00:00:00Z
    user: username
    virtual_host: domain.tld
  -
    domain: www.domain.tld
    order_item_id: 000000000
    request_time: 0000-00-00T00:00:00Z
    user: username
    virtual_host: domain.tld
  -
    domain: mail.domain.tld
    order_item_id: 000000000
    request_time: 0000-00-00T00:00:00Z
    user: username
    virtual_host: domain.tld
```
Available in cPanel 62+: [WHM API 1 Functions - get_autossl_pending_queue](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_pending_queue)
* `whmapi1 get_autossl_pending_queue_for_domain`: Outputs enqueued AutoSSL requested associated with a specific domain name.
```
queue_by_domain:
  -
    domain: domain.tld
    last_poll_time: 0000-00-00T00:00:00Z
    order_item_id: 000000000
	request_time: 0000-00-00T00:00:00Z
    vhost_name: domain.tld
  -
    domain: www.domain.tld
    last_poll_time: 0000-00-00T00:00:00Z
    order_item_id: 000000000
	request_time: 0000-00-00T00:00:00Z
    vhost_name: domain.tld
  -
    domain: mail.domain.tld
    last_poll_time: 0000-00-00T00:00:00Z
    order_item_id: 000000000
	request_time: 0000-00-00T00:00:00Z
    vhost_name: domain.tld
```
Available in cPanel 68+: [WHM API 1 Functions - get_autossl_pending_queue_for_domain](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_pending_queue_for_domain)
* `whmapi1 get_autossl_pending_queue_for_user`: Outputs enqueued AutoSSL requests associated with a specific username. Output is the same as `whmapi1 get_autossl_pending_queue_for_domain`.  
Available in cPanel 68+: [WHM API 1 Functions - get_autossl_pending_queue_for_user](https://documentation.cpanel.net/display/SDK/WHM+API+1+Functions+-+get_autossl_pending_queue_for_user)

### UAPI
All of the following UAPI calls that will be reviewed will be using YAML formatting. UAPI also supports XML & JSON formatting which should be taken into consideration when executing these commands.
This section does not cover all AutoSSL UAPI calls. It will only cover the needed commands that are needed to fulfill their GUI counterparts.

#### cPanel: SSL/TLS Status
* `uapi SSL add_autossl_excluded_domains`: Exclude domains under the user's account from AutoSSL.  
Available in cPanel 66+: [UAPI Functions - SSL::add_autossl_excluded_domains](https://documentation.cpanel.net/display/SDK/UAPI+Functions+-+SSL%3A%3Aadd_autossl_excluded_domains)
* `uapi SSL remove_autossl_excluded_domains`: Remove a domain that is excluded from AutoSSL.  
Available in cPanel 66+: [UAPI Functions - SSL::remove_autossl_excluded_domains](https://documentation.cpanel.net/display/SDK/UAPI+Functions+-+SSL%3A%3Aremove_autossl_excluded_domains)
* `uapi SSL set_autossl_excluded_domains`: Replaces the current excluded domains with new ones.  
Available in cPanel 66+: [UAPI Functions - SSL::set_autossl_excluded_domains](https://documentation.cpanel.net/display/SDK/UAPI+Functions+-+SSL%3A%3Aset_autossl_excluded_domains)
* `uapi SSL get_autossl_excluded_domains`: Returns domains that are excluded from AutoSSL.  
```
    -
      excluded_domain: domain.tld
```
Available in cPanel 66+: [UAPI Functions - SSL::get_autossl_excluded_domains](https://documentation.cpanel.net/display/SDK/UAPI+Functions+-+SSL%3A%3Aget_autossl_excluded_domains)

### Backend
This section will be covering the navigation of AutoSSL through its backend files and programs.

#### WHM: Providers
* `/etc/cron.d/cpanel_autossl`: Cron used to schedule an AutoSSL check on all users.
* `/var/cpanel/autossl.json` contains the current AutoSSL provider.

#### WHM: Options
* `/var/cpanel/autossl.json` contains metadata including whether or not AutoSSL will replace certificates it did not issue.

#### WHM: Logs
* Logs for AutoSSL are timestamped and saved to directories within `/var/cpanel/logs/autossl`.
* `/var/cpanel/autossl_problems.sqlite`

#### WHM: Manage Users
* `/usr/local/cpanel/bin/autossl_check` is used to run an AutoSSL check for one or all users.
* cPanel user files stored within `/var/cpanel/users` will determine whether or not AutoSSL is **manually** enabled or disabled.
 * This is determined using the variable `FEATURE-AUTOSSL`.

#### WHM: Pending Queue
* `/usr/local/cpanel/bin/autossl_check_cpstore_queue` will check the status of cPanel-issued certificates that are currently enqueued.
* `/etc/cron.d/cpanel_autossl_cpanel_provider` checks the AutoSSL queue using `/usr/local/cpanel/bin/autossl_check_cpstore_queue`.
* `/var/cpanel/autossl_queue_cpanel.sqlite` is a SQLite database that stores the current AutoSSL queue.

#### cPanel: SSL/TLS Status
* AutoSSL exclusions are saved on a per-user basis to the directory `/var/cpanel/ssl/autossl/excludes`.
```
{
    "excluded_domains": [
        "domain.tld"
    ]
}
```

# Notifications
A separate document detailing how to manage notifications is [available here](https://github.com/Sartron/documents/blob/master/cP_autossl/notifications.md).

# Troubleshooting
Troubleshooting AutoSSL is stored in a separate document. Please see [troubleshooting.md](https://github.com/Sartron/documents/blob/master/cP_autossl/troubleshooting.md).

# Redirects
* AutoSSL's required DCV checks will fail if a redirect is encountered in cPanel 58, the debut version for AutoSSL.
* As of cPanel 60, using the **cPanel » Domains » Redirects** menu will also append directives so that the DCV check is excluded from the redirect.
* These are the current directives that are to be used to avoid redirect issues:
  * cPanel Internal DCV
  ```
  RewriteCond %{REQUEST_URI} !^/[0-9]+\..+\.cpaneldcv$
  ```
  * Comodo DCV
  ```
  RewriteCond %{REQUEST_URI} !^/\.well-known/pki-validation/[A-F0-9]{32}\.txt(?:\ Comodo\ DCV)?
  ```
  * Let's Encrypt DCV
  ```
  RewriteCond %{REQUEST_URI} !^/\.well-known/acme-challenge/[0-9a-zA-Z_-]+$
  ```
* cPanel 64 includes a setting within **WHM » Server Configuration » Tweak Settings » Domains** that can be used to set these redirects up within the Apache configuration file.
  * The name of the setting currently varies between cPanel 64 and 66. I have added both names below:
    * cPanel 64: _Use a Global DCV rewrite exclude instead of .htaccess modification (requires Apache 2.4+, EA4)_
	* cPanel 66: _Use a Global DCV Passthrough instead of .htaccess modification (requires EA4)_
  * This setting will not appear if EasyApache 4 is not installed on the server.
  * This will add rewrite rules before the virtual hosts begin. The example provided will also include the rewrite for Let's Encrypt which is only available with the Let's Encrypt plugin installed:
  ```
  <IfModule rewrite_module>
  # Global DCV Exclude
  RewriteEngine on
  RewriteCond %{REQUEST_URI} ^/\.well-known/acme-challenge/[0-9a-zA-Z_-]+$ [OR]
  RewriteCond %{REQUEST_URI} ^/\.well-known/pki-validation/[A-F0-9]{32}\.txt(?:\ Comodo\ DCV)?$ [OR]
  RewriteCond %{REQUEST_URI} ^/[0-9]+\..+\.cpaneldcv$
  
  # Exclude proxy subdomains as we need rewrites to capture the DCV requests
  RewriteCond %{HTTP_HOST} !^(?:autoconfig|autodiscover|cpanel|cpcalendars|cpcontacts|webdisk|webmail|whm)\.
  RewriteRule ^ - [END]
  </IfModule>
  ```
  * The virtual host for the server hostname will also include its own rewrite rules. The example provided will also include the rewrite for Let's Encrypt which is only available with the Let's Encrypt plugin installed:
  ```
  RewriteEngine On
  RewriteCond %{REQUEST_URI} ^/[0-9]+\..+\.cpaneldcv$ [OR]
  RewriteCond %{REQUEST_URI} ^/\.well-known/acme-challenge/[0-9a-zA-Z_-]+$ [OR]
  RewriteCond %{REQUEST_URI} ^/\.well-known/pki-validation/[A-F0-9]{32}\.txt(?:\ Comodo\ DCV)?$
  RewriteRule ^ /.cpanel/dcv [passthrough]
  ```
* Citations
  * [AutoSSL and HTTP Redirects](https://forums.cpanel.net/threads/567801/)
  * [Tweak Settings - Domains](https://documentation.cpanel.net/display/64Docs/Tweak+Settings+-+Domains#TweakSettings-Domains-UseaGlobalDCVrewriteexcludeinsteadof.htaccessmodification(requiresApache2.4andEasyApache4))

# Let's Encrypt
As of cPanel 58.0.17, Let's Encrypt was introduced as an official provider for AutoSSL certificates.

## Installing & Uninstalling
cPanel does not natively come with this provider built in, but has to be installed manually by running `/usr/local/cpanel/scripts/install_lets_encrypt_autossl_provider`.  
Once installed, the user must agree to the terms of service provided by Let's Encrypt. Here is what that looks like within `/var/cpanel/autossl.json`:
``` 
{
    "_schema_version": 1,
    "provider": "LetsEncrypt",
    "provider_properties": {
        "LetsEncrypt": {
            "terms_of_service_accepted": "https://acme-v01.api.letsencrypt.org/terms"
        }
    }
}
```
If the Let's Encrypt provider is no longer needed and you wish to uninstall it, run `/usr/local/cpanel/scripts/uninstall_lets_encrypt_autossl_provider`.

## General
* The primary positive factor of Let's Encrypt is that certificates are issued nearly instantly.
* Let's Encrypt also does not impose restrictions on the content of the site or origin of the site. This makes it a useful alternative for when Comodo is rejecting certificate requests.
* DCV for Let's Encrypt is performed in ./.well-known/acme-challenge.
* The AutoSSL implementation of Let's Encrypt does have greater restrictions in comparison to Comodo when comparing the amount and rate of certificates that can be issued per virtual host.
* The following is documentation on how Let's Encrypt technically works: [Automatic Certificate Management Environment (ACME)](https://github.com/ietf-wg-acme/acme/)

# Limitations
* Domain/Rate Limitations:
  * AutoSSL can only secure up to 200 domains per virtual host block, including domains prefixed by www.. If the domain does not include www., then AutoSSL will automatically secure it as well.
  * Let's Encrypt can only secure up to 100 domains per virtual host block.
* SSLs issued by the provider Comodo are subject to being rejected. This is possible if the site contains the name of a country that is restricted by US Export laws.