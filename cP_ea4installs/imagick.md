# Overview
This document details the installation process for the PHP extension **imagick** on a cPanel EasyApache 4 server.

# Installation
1. Install ImageMagick-devel if not already installed: `rpm -q ImageMagick-devel || yum -y install ImageMagick-devel`
2. Install imagick using PECL: `pecl install imagick`

# Errors
You may also need to install pcre-devel: `rpm -q pcre-devel || yum -y install pcre-devel`