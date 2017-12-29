# Overview
This document details the installation process for the Apache module **mod_cloudflare** on a cPanel EasyApache 4 server.

# Installation
1. Install the RPM `ea-apache24-devel`, which provides the **apxs** utility.
2. Download the source file for mod_cloudflare from https://www.cloudflare.com/static/misc/mod_cloudflare/mod_cloudflare.c.
3. Build the module from source using `apxs -a -i -c mod_cloudflare.c`.
```
[root@server src]# apxs -a -i -c mod_cloudflare.c
/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool --silent --mode=compile gcc -std=gnu99 -prefer-pic -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic  -DLINUX -D_REENTRANT -D_GNU_SOURCE -pthread -I/usr/include/apache2  -I/opt/cpanel/ea-apr15/include/apr-1   -I/opt/cpanel/ea-apr15/include/apr-1   -c -o mod_cloudflare.lo mod_cloudflare.c && touch mod_cloudflare.slo
mod_cloudflare.c: In function ‘set_cf_default_proxies’:
mod_cloudflare.c:187: warning: initialization from incompatible pointer type
/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool --silent --mode=link gcc -std=gnu99 -Wl,-z,relro,-z,now   -o mod_cloudflare.la  -rpath /usr/lib64/apache2/modules -module -avoid-version    mod_cloudflare.lo
/usr/lib64/apache2/build/instdso.sh SH_LIBTOOL='/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool' mod_cloudflare.la /usr/lib64/apache2/modules
/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool --mode=install install mod_cloudflare.la /usr/lib64/apache2/modules/
libtool: install: install .libs/mod_cloudflare.so /usr/lib64/apache2/modules/mod_cloudflare.so
libtool: install: install .libs/mod_cloudflare.lai /usr/lib64/apache2/modules/mod_cloudflare.la
libtool: install: install .libs/mod_cloudflare.a /usr/lib64/apache2/modules/mod_cloudflare.a
libtool: install: chmod 644 /usr/lib64/apache2/modules/mod_cloudflare.a
libtool: install: ranlib /usr/lib64/apache2/modules/mod_cloudflare.a
libtool: finish: PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/opt/cpanel/composer/bin:/root/bin:/sbin" ldconfig -n /usr/lib64/apache2/modules
----------------------------------------------------------------------
Libraries have been installed in:
   /usr/lib64/apache2/modules

If you ever happen to want to link against installed libraries
in a given directory, LIBDIR, you must either use libtool, and
specify the full pathname of the library, or use the `-LLIBDIR'
flag during linking and do at least one of the following:
   - add LIBDIR to the `LD_LIBRARY_PATH' environment variable
     during execution
   - add LIBDIR to the `LD_RUN_PATH' environment variable
     during linking
   - use the `-Wl,-rpath -Wl,LIBDIR' linker flag
   - have your system administrator add LIBDIR to `/etc/ld.so.conf'

See any operating system documentation about shared libraries for
more information, such as the ld(1) and ld.so(8) manual pages.
----------------------------------------------------------------------
chmod 755 /usr/lib64/apache2/modules/mod_cloudflare.so
[activating module `cloudflare' in /etc/apache2/conf.modules.d/mod_cloudflare.conf]
```
4. The module has been built and should automatically be enabled in Apache's configuration. This can be verified using `apachectl -M`.