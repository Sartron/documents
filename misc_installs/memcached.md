# Overview
This document details the installation process for the service **memcached** as well as its available PHP extensions.  
It is recommended to perform PHP extensions using PECL, which is available from the PEAR RPM.

# Installation
Some pointers to address before proceeding with any installations:
* Be careful not to confuse the PHP extensions memcache**d** and memcache. memcached is built on the [libmemcached library](http://libmemcached.org/libMemcached.html) and still remains updated to this day, whereas memcache is several years old and no longer updated.  
* You must have the memcached service installed on the server in order to actually _utilize_ its respective PHP extensions.
* If your PHP installation is installed as an SCL, use `scl enable` when performing any PECL installation. Alternatively, use the absolute path for the PECL binary you are referencing.

## memcached **service**
**CentOS 6**: `yum install memcached; chkconfig memcached on`  
**CentOS 7**: `yum install memcached; systemctl enable memcached; systemctl start memcached`

### Configuration
`/etc/sysconfig/memcached`  

By default, memcached listens on port 11211. You **should not** expose this port to remote connections as memcached does not take comprehensive precautions in authenticating access.  
Further documentation on memcached is available [here](https://github.com/memcached/memcached/wiki).

## memcache **PHP extension**
`pecl install memcache`  
This PHP extension is not recommended, and instead it is recommended to install memcache**d**.

## memcached **PHP extension**
**PHP <7.0**: `yum install libmemcached-devel; pecl install memcached-2.2.0`  
**PHP >=7.0**: `yum install libmemcached-devel; pecl install memcached`

### EasyApache 4 - Looped SCL Installation
An example of a basic script that could install the PHP extension memcached on an EasyApache 4 server.
```
yum -y install libmemcached-devel

for scl in $(scl -l)
do
	grep 'ea-php5' <<< ${scl} && scl enable ${scl} 'pecl install memcached-2.2.0';
	grep 'ea-php7' <<< ${scl} && scl enable ${scl} 'pecl install memcached';
done
```