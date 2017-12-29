# Notifications
This section will be covering the configuration of notifications for AutoSSL. These notifications are available as of **[cPanel 68](https://documentation.cpanel.net/display/68Docs/68+Release+Notes#id-68ReleaseNotes-SSLandAutoSSLcertificaterenewal,expiry,failure,andsuccessnotifications)**.

## WHM
When configuring notifications for AutoSSL from WHM, please note that these settings determines whether or not the setting will be **visible** from cPanel.  
This does _not_ mean disabling the setting in WHM will automatically disable it for all users that had the setting enabled.  
When reading, please note that the term '_Setting Name_' will be used to match WHM settings with their cPanel counterparts.

![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/whm_options.png)
AutoSSL notifications for WHM are configured from _WMH » SSL/TLS » Manage AutoSSL » Options_. The file-based location of this configuration is `/var/cpanel/autossl.json`.
* Notify when AutoSSL defers certificate renewal because a domain on the current certificate has failed DCV.
    * Setting Name: `notify_autossl_expiry_coverage`
* Notify when AutoSSL will not secure new domains because a domain on the current certificate has failed DCV.
    * Setting Name: `notify_autossl_renewal_coverage`
* Send notifications when AutoSSL has renewed a certificate.
    * Setting Name: `notify_autossl_renewal`

_WHM » Server Configuration » Tweak Settings » Notifications_ supports an additional option in regards to AutoSSL notifications. The file-based location of this configuration is `/var/cpanel/cpanel.config`.
* Send notifications when certificates approach expiry.  
_Send a notification when an SSL certificate expires soon. The system will only send a notification for an AutoSSL-provided certificate if that certificate fails to renew._
    * Setting Name: `notify_autossl_expiry`

If the previously aforementioned setting is **enabled**, you may also configure the following setting within _WHM » Server Contacts » Contact Manager » Notifications_. Please remember that these settings are for the system e-mail, not for cPanel users.
* AutoSSL certificates expiring  
_This option indicates that an AutoSSL certificate will expire soon._  

## cPanel
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_autossl/images/cP_notifications.png)
AutoSSL notifications for cPanel are configured from _cPanel » Contact Information_. Alternatively, they can be edited using [cPanel API 2 Functions - CustInfo::savecontactinfo](https://documentation.cpanel.net/display/SDK/cPanel+API+2+Functions+-+CustInfo%3A%3Asavecontactinfo). The file-based location of this configuration is stored in username files in `/var/cpanel/users`.  

Even if AutoSSL is disabled, the user can still view the AutoSSL notification settings. However, in order to actually **change** the settings, they must have AutoSSL enabled.  

The following settings are available at the moment:
* AutoSSL has renewed a certificate.  
_The system will notify you when it has installed an AutoSSL certificate._
	* Setting Name: `notify_autossl_renewal`
* AutoSSL will not secure a new domain because a domain on the current certificate has failed DCV.
	* Setting Name: `notify_autossl_renewal_coverage`
* AutoSSL defers certificate renewal because a domain on the current certificate has failed DCV.
	* Setting Name: `notify_autossl_expiry_coverage`
* AutoSSL certificate expiry.  
_The system will notify you if an AutoSSL certificate will expire soon._
	* Setting Name: `notify_autossl_expiry`