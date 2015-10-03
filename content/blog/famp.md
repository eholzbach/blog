---
date: "2011-12-21T21:15:00-07:00"
title: "FreeBSD, Apache, MySQL, PHP = FAMP"
slug: "freebsd-apache-mysql-php-famp"
---

A simple script to install Apache, MySQL and PHP on FreeBSD with userdir and server-status enabled. Warning, this may or may not be a sane configuration.

Due to a [lawsuit from Bell Labs in the the 90's](http://en.wikipedia.org/wiki/USL_v._BSDi) your peers are saying "[LAMP](http://en.wikipedia.org/wiki/LAMP_(software_bundle))" instead of "[FAMP](./famp.sh)". The latter acronym is far more hilarious. Please blurt it out at any chance you get.

While it looks like this, need to [download it from here](./famp.sh) for it to work due to line breaks. I typed "please test before using in production". This has been redacted. If you're deploying FAMP stacks you already have a solution for implementation.


#!/bin/sh
# quick FAMP stack for new systems
# installs apache22, php5, and mysql55 on freebsd
# binds to first ip on interface
# use 'setenv BATCH YES' before running

if [ -f /usr/ports/.portsnap.INDEX ]; then
cd /usr/ports ;portsnap fetch update
else
cd /usr/ports ;portsnap fetch extract
fi

cd /usr/ports/www/apache22 ;make config
cd /usr/ports/lang/php5 ;make config
cd /var/db/ports/php5
sed -i "" 's/WITHOUT_APACHE=true/WITH_APACHE=true/' /var/db/ports/php5/options
cd /usr/ports/databases/mysql55-server ;make config

cd /usr/ports/www/apache22 ;make install clean
