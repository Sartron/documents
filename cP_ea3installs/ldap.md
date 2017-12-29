# Overview
This document details the installation process for the PHP extension **ldap** on a cPanel EasyApache 3 server.

# Installation
1. Install the RPM `openldap-devel`. This provides the needed source files such as `/usr/include/ldap.h` required to build the ldap extension.
2. Add the argument `--with-ldap` to `/var/cpanel/easy/apache/rawopts/all_php5`.
3. Build EasyApache 3 using `/usr/local/cpanel/scripts/easyapache`.
4. PHP has been recompiled with ldap successfully. This can be verified using `php -m`.