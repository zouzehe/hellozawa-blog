 ```
 ./configure  --prefix=/usr/local/php7.4 --exec-prefix=/usr/local/php7.4 --bindir=/usr/local/php7.4/bin --sbindir=/usr/local/php7.4/sbin --includedir=/usr/local/php7.4/include --libdir=/usr/local/php7.4/lib/php --mandir=/usr/local/php7.4/php/man --with-config-file-path=/usr/local/php7.4/etc --with-mhash --with-openssl --with-mysqli=shared,mysqlnd --with-pdo-mysql=shared,mysqlnd --with-iconv=/usr/local/lib --with-zlib --with-zip --enable-inline-optimization --disable-debug --disable-rpath --enable-shared --enable-xml --enable-bcmath --enable-shmop --enable-fileinfo --enable-sysvsem --enable-mbregex --enable-mbstring --enable-ftp --enable-pcntl --enable-sockets --with-xmlrpc --enable-soap --without-pear --with-gettext --enable-session --with-curl --enable-gd --with-webp --with-jpeg=/usr/local/libjpeg --with-freetype=/usr/local/freetype --enable-cli --with-libxml --with-xpm --enable-opcache --with-pdo-sqlite --enable-exif --enable-fpm --with-fpm-user=nginx --with-fpm-group=nginx --without-gdbm PKG_CONFIG_PATH=/usr/local/lib/pkgconfig/
 
 wget https://www.sqlite.org/2020/sqlite-autoconf-3320300.tar.gz
wget http://ftp.gnu.org/pub/gnu/libiconv/libiconv-1.14.tar.gz

wget https://github.com/swoole/swoole-src/archive/v4.5.2.tar.gz

wget https://libzip.org/download/libzip-1.2.0.tar.gz


rpm -qf /usr/bin/autoconf
rpm -e --nodeps autoconf-2.63
wget ftp://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz

./configure \
--with-php-config=/usr/local/php7.4/bin/php-config \
--enable-openssl \
--enable-http2 

wget https://pecl.php.net/get/mongodb-1.7.5.tgz



prefix=/usr/local
exec_prefix=${prefix}
libdir=${exec_prefix}/lib
includedir=${prefix}/include

Name: SQLite
Description: SQL database engine
Version: 3.32.3
Libs: -L${libdir} -lsqlite3
Libs.private: -lz -lm -ldl -lpthread
Cflags: -I${includedir}


 ```
 
