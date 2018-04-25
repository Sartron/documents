# Overview
This article will be covering the installation process of **Subversion** and **mod_dav_svn** on a cPanel server.

# Subversion Installation
Subversion can be installed from the base CentOS/CloudLinux repos. Use the following command to install this RPM:
```
yum -y install subversion
```

# mod_dav_svn Installation
**Download** and extract the RPM `mod_dav_svn`.
```
yumdownloader mod_dav_svn
rpm2cpio mod_dav_svn*.rpm | cpio -id
```

Move the extracted files into their respective directories.
```
mv ./etc/httpd/conf.modules.d/10-subversion.conf /etc/apache2/conf.modules.d/
mv ./usr/lib64/httpd/modules/*.so /etc/apache2/modules/
mv ./usr/share/doc/mod_dav_svn* /usr/share/doc/
```

Restart Apache and verify that the module is installed.
```
apachectl -M | grep 'dav_svn'
```

# Errors
* You will get this error when checking the Apache syntax using `apachectl -t` or when restarting the service. This means that library `/lib64/libsvn_repos-1.so.0` is missing.
```
Cannot load modules/mod_dav_svn.so into server: libsvn_repos-1.so.0: cannot open shared object file: No such file or directory
```
Resolve this issue by installing the needed libraries using `yum -y install subversion-libs`.
* You will get this error when checking the Apache syntax using `apachectl -t` or when restarting the service.
````
Cannot load modules/mod_dav_svn.so into server: /etc/apache2/modules/mod_dav_svn.so: undefined symbol: dav_do_find_liveprop
```
Ensure that the Apache module mod_dav is installed and is getting **loaded before** mod_dav_svn is.
 * EasyApache 4 natively supports mod_dav. Install this module using the following command:
 ```
 yum -y install ea-apache24-mod_dav
 ```
 * This is an example of changing the load order of modules. In my example, mod_dav is loaded from the file `/etc/apache2/conf.modules.d/150_mod_dav.conf` so I renamed the file loading mod_dav_svn to 160 so it is loaded afterwards.
 ```
 mv /etc/apache2/conf.modules.d/10-subversion.conf /etc/apache2/conf.modules.d/160-subversion.conf
 ```
* If you get an error about missing apr libraries, you will need to ensure that RPMS ea-apr-util and ea-apr are installed.
```
yum -y install ea-apr-util ea-apr
ln -s /opt/cpanel/ea-apr16/lib64/libaprutil-1.so.0 /lib64/libaprutil-1.so.0
ln -s /opt/cpanel/ea-apr16/lib64/libapr-1.so.0 /lib64/libapr-1.so.0
```