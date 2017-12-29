# Installing an SSL certificate in WHM
##### Accessing the interface:
Login to WHM as root and navigate to _WHM » SSL/TLS » Install an SSL Certificate on a Domain_. You may use the search bar or access the SSL/TLS section and click the button for it.
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_sslinstall/whm_search_shadow.png)

##### Installing:
On the next page there will be a few empty fields that you will need to fill out. These fields will correspond to the needed information for the SSL certificate you are installing. Whenever you fill out a field correctly, cPanel will present you a green arrow indicating this: ![](https://raw.githubusercontent.com/Sartron/documents/master/cP_sslinstall/whm_check.png)
1. Start by filling out the **Domain** field with the domain name you are installing the SSL for.
2. In most scenarios, you should not fill out the **IP Address** field. cPanel will automatically retrieve the IP for the domain you are installing the SSL over.
3. Next, fill out the **Certificate** field. You will want to copy and paste the SSL certificate into this field. An SSL certificate contains the following text:
    ```
    -----BEGIN CERTIFICATE-----
    -----END CERTIFICATE-----
    ```
4. Fill out the **Private Key** field. A private key contains the following text:
    ```
    -----BEGIN RSA PRIVATE KEY-----
    -----END RSA PRIVATE KEY-----
    ```
5. Fill out the the **Certificate Authority Bundle**. In most scenarios, cPanel will not require this field however it is highly recommended to provide this if you have it. This bundle is composed of the root or intermediate certificates that should accompany the base certificate.  

While performing the installation, you may come across several buttons that may assist you in this installation.  
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_sslinstall/whm_buttons.png)  

Please note that these buttons can only extract information that already exists on the server. This means you cannot autofill a field (e.g. certificate or private key) that did not already exist on the server. This makes these buttons more useful for renewals or certificates that need to be re-installed.
* Browse Certificates: Pulls up a separate UI for browsing certificates that are present in the SSL Storage for the server. Selecting one of these certificates will automatically populate all fields in the installation menu.
* Autofill by Domain: Autofills the remaining fields based on an SSL certificate that already exists for the provided domain name. This will not be used for a fresh installation.
* Autofill by Certificate: Autofills the remaining fields based on the SSL certificate provided.

##### Post-installation:
The server will output out some information on the SSL. If the installation was successful the message _SSL Host Successfully Installed_ should be displayed.  
![](https://raw.githubusercontent.com/Sartron/documents/master/cP_sslinstall/whm_installed_shadow.png)