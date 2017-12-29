# Overview
This document details the installation process for the PHP extension **imagick** on a cPanel EasyApache 4 server.

# Installation
1. Install and enable the EPEL RPM `epel-release` if not done already. This will provide access to the GeoIP RPMs within the active yum repositories.
2. Install the RPMs `GeoIP` and `GeoIP-devel`. The latter is needed for GeoIP to actually work, and the former is needed to build GeoIP.
3. Download and extract the latest archive of GeoIP available from the [GitHub repo](https://github.com/maxmind/geoip-api-mod_geoip2/releases).
4. Build mod_geoip2 using **apxs**. This process is detailed in the aforementioned [Github repo](https://github.com/maxmind/geoip-api-mod_geoip2/blob/master/INSTALL.md).
```
[root@server geoip-api-mod_geoip2-1.2.10]# apxs -i -a -L/usr/lib64 -I/usr/include -lGeoIP -c mod_geoip.c
/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool --silent --mode=compile gcc -std=gnu99 -prefer-pic -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic  -DLINUX -D_REENTRANT -D_GNU_SOURCE -pthread -I/usr/include/apache2  -I/opt/cpanel/ea-apr15/include/apr-1   -I/opt/cpanel/ea-apr15/include/apr-1  -I/usr/include  -c -o mod_geoip.lo mod_geoip.c && touch mod_geoip.slo
/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool --silent --mode=link gcc -std=gnu99 -Wl,-z,relro,-z,now   -o mod_geoip.la  -L/usr/lib64 -lGeoIP -rpath /usr/lib64/apache2/modules -module -avoid-version    mod_geoip.lo
/usr/lib64/apache2/build/instdso.sh SH_LIBTOOL='/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool' mod_geoip.la /usr/lib64/apache2/modules
/opt/cpanel/ea-apr15/lib64/apr-1/build/libtool --mode=install install mod_geoip.la /usr/lib64/apache2/modules/
libtool: install: install .libs/mod_geoip.so /usr/lib64/apache2/modules/mod_geoip.so
libtool: install: install .libs/mod_geoip.lai /usr/lib64/apache2/modules/mod_geoip.la
libtool: install: install .libs/mod_geoip.a /usr/lib64/apache2/modules/mod_geoip.a
libtool: install: chmod 644 /usr/lib64/apache2/modules/mod_geoip.a
libtool: install: ranlib /usr/lib64/apache2/modules/mod_geoip.a
libtool: finish: PATH="/usr/local/jdk/bin:/usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/opt/cpanel/composer/bin:/usr/local/easy/bin:/usr/local/bin:/usr/X11R6/bin:/usr/local/puppet/sbin:/usr/local/puppet/bin:/root/bin:/sbin" ldconfig -n /usr/lib64/apache2/modules
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
chmod 755 /usr/lib64/apache2/modules/mod_geoip.so
[activating module `geoip' in /etc/apache2/conf.modules.d/mod_geoip.conf]
```
5. The module has been built and should automatically be enabled in Apache's configuration. This can be verified using `apachectl -M`.

# Configuration
Once the installation is completed, you may want to configure GeoIP and enable it server-wide. [This page](http://dev.maxmind.com/geoip/legacy/mod_geoip2/#Configuration) details how to configure the module.