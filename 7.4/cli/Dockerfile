FROM phusion/baseimage:bionic-1.0.0

ENV GNU_ARCH="x86_64-linux-gnu"
ENV DEB_MULTIARCH="x86_64-linux-gnu"
ENV SRC_DIR="/usr/src"
ENV PHP_SRC_DIR="/usr/src/php"

# dependencies required for running "phpize"
ENV PHPIZE_DEPS \
		autoconf \
		dpkg-dev \
		file \
		g++ \
		gcc \
		libc-dev \
		make \
		pkg-config \
		re2c

# persistent / runtime deps
ENV PERSISTENT_DEPS \
		ca-certificates \
		curl \
		xz-utils \
		gnupg \
		dirmngr 

ENV PHP_INI_DIR="/etc/php"

RUN set -eux \
\
	# Prevent Debian's PHP packages from being installed at all
	&& printf "Package: php*\nPin: release *\nPin-Priority: -1" > /etc/apt/preferences.d/no-debian-php \
	# Ensure we have the PHP INI Dir we desire in place.
	&& mkdir -p "$PHP_INI_DIR/conf.d" \
	# Allow running as an arbitrary user
	&& mkdir -p /var/www/html \
	&& chown www-data:www-data /var/www/html \
	&& chmod 777 /var/www/html \
	# Create a src directory to work in
	&& mkdir -p $SRC_DIR; 

## Handle Some Apt Stuff
RUN set -eux \
\
	&& apt-get update \
	# Install the Apt Packages for Persistent / Runtime Deps
	&& apt-get install -y --no-install-recommends \
		$PHPIZE_DEPS \
		$PERSISTENT_DEPS \
	; 
	
# Apply stack smash protection to functions using local buffers and alloca()
# Make PHP's main executable position-independent (improves ASLR security mechanism, and has no performance impact on x86_64)
# Enable optimization (-O2)
# Enable linker optimization (this sorts the hash buckets to improve cache locality, and is non-default)
# https://github.com/docker-library/php/issues/272
# -D_LARGEFILE_SOURCE and -D_FILE_OFFSET_BITS=64 (https://www.php.net/manual/en/intro.filesystem.php)
ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"
ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O1 -pie"

ENV GPG_KEYS 1729F83938DA44E27BA0F4D3DBDB397470D12172 BFDDD28642824F8118EF77909B67A5C12229118F

ENV PHP_VERSION 7.4.13
ENV PHP_URL="https://www.php.net/distributions/php-7.4.13.tar.xz" 
ENV PHP_SHA256="aead303e3abac23106529560547baebbedba0bb2943b91d5aa08fff1f41680f4"

# Download and Verify and Extract the Package
WORKDIR $SRC_DIR
RUN set -eux \
\
	&& curl -fsSL -o php.tar.xz "$PHP_URL"  \
\
	# Ensure the expected Checksum
	&& echo "$PHP_SHA256 php.tar.xz" | sha256sum -c - \
\
	&& mkdir -p $PHP_SRC_DIR \
	&& tar -Jxf php.tar.xz -C "$PHP_SRC_DIR" --strip-components=1; 

ENV PHP_BUILD_DEPS \
		libargon2-0-dev \
		libcurl4-openssl-dev \
		libedit-dev \
		libonig-dev \
		libwebp-dev \
		libsodium-dev \
		libsqlite3-dev \
		libssl-dev \
		libxml2-dev \
		zlib1g-dev \
		libpng-dev \
		libicu-dev \
		libzip-dev \
		libgd-dev \
		libjpeg-turbo8-dev \
		postgresql \
		postgresql-contrib \
		libpq-dev \
		libmysqlclient-dev \
		libreadline-dev \
		libbz2-dev \
		libxslt1-dev \
		unixodbc-dev
		
# # Lets build this sucker
WORKDIR $PHP_SRC_DIR
RUN set -eux \
\
    # Install the Apt Packages for Build Deps	
	&& apt-get install -y --no-install-recommends $PHP_BUILD_DEPS \
	&& ln -sT "/usr/include/$DEB_MULTIARCH/curl" /usr/local/include/curl

RUN set -eux \
\
	&& export \
		CFLAGS="$PHP_CFLAGS" \
		CPPFLAGS="$PHP_CPPFLAGS" \
		LDFLAGS="$PHP_LDFLAGS" \
	# Configure that sucker
	&& ./configure \
		--build=$GNU_ARCH \
		--with-config-file-path="$PHP_INI_DIR" \
		--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
		# make sure invalid --configure-flags are fatal errors instead of just warnings
		--enable-option-checking=fatal \
		--enable-debug \
		# enable-ftp is included here because ftp_ssl_connect() needs ftp to be compiled statically 
		--enable-ftp  \
		--with-pcre-jit \
		--with-webp  \
		--with-jpeg  \
		--with-mhash \
		--with-mysql-sock  \
		--with-libedit \
		# https://wiki.php.net/rfc/argon2_password_hash (7.2+)
		--with-password-argon2  \
		--with-pic \
		# Build as shared modules (so that they can be replaced later if so desired)
		--enable-bcmath=shared  \
		--with-bz2=shared \
		--enable-calendar=shared \
		--with-curl=shared \
		--enable-exif=shared  \
		--enable-gd=shared \
		--with-gettext=shared \
		--enable-intl=shared \
		--enable-mbstring=shared \
		--with-mysqli=shared \
		--enable-mysqlnd=shared \
		--with-unixODBC=shared,/usr \
		--enable-opcache=shared \
		--with-openssl=shared \
		--enable-pcntl=shared \
		--with-pdo-mysql=shared  \
		--with-pdo-pgsql=shared \
		--with-pgsql=shared \
		--with-readline=shared \
		--enable-soap=shared \
		--enable-sockets=shared \
		# https://wiki.php.net/rfc/libsodium
		--with-sodium=shared \
		# always build against system sqlite3 (https://github.com/php/php-src/commit/6083a387a81dbbd66d6316a3a12a63f06d5f7109)
		--with-sqlite3=shared,/usr \
		--with-pdo-sqlite=shared,/usr  \
\
		--with-xsl=shared \
		--with-zip=shared \
		--with-zlib=shared \
		# in PHP 7.4+, the pecl/pear installers are officially deprecated (requiring an explicit "--with-pear")
		--with-pear \
		--with-libdir="lib/$DEB_MULTIARCH" \
	&& make -j "$(nproc)" \
	&& find -type f -name '*.a' -delete \
	&& make install \
	&& find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true \
	&& make clean \
\
	# Copy default example "php.ini" files somewhere easily discoverable
	&& cp -v php.ini-* "$PHP_INI_DIR/" \
	\
	# smoke test
	&& php --version

# Create configuration files for all of those shared modules.
WORKDIR /usr/local/lib/php/extensions/debug-non-zts-20190902
RUN set -eux \
\
	&& mkdir -p /etc/php/mods-available \
	&& for f in *.so ; do printf "; configuration for php ${f%.so} module\n; priority=20\nextension=$f" > /etc/php/mods-available/${f%.so}.ini ; done


# Install Composer
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer


RUN for extension in zip zlib mbstring openssl; do ln -sT /etc/php/mods-available/$extension.ini /etc/php/conf.d/$extension.ini; done

WORKDIR /tmp

# Install Pickle
ENV PICKLE_VERSION 0.6.0
ENV PICKLE_URL="https://github.com/FriendsOfPHP/pickle/releases/download/v0.6.0/pickle.phar" 
ENV PICKLE_SHA256="0bffcbff8368b85fcf30b098b901067c5cea2abc71c4c907c836dbb8c6f91f55"

RUN set -eux \
\
	&& curl -fsSL -o pickle.phar "$PICKLE_URL" \
	# Ensure the expected Checksum
	&& echo "$PICKLE_SHA256 pickle.phar" | sha256sum -c - \
	&& mv pickle.phar /usr/local/bin/pickle \
	&& chmod +x /usr/local/bin/pickle

RUN set -eux \
\
	&& curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add - \
	&& curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > /etc/apt/sources.list.d/mssql-release.list \
	&& apt update \
	&& ACCEPT_EULA=Y apt install -y --no-install-recommends \
			msodbcsql17 \
			mssql-tools \
			unixodbc-dev \
			libyaml-dev \
	&& ln -sfn /opt/mssql-tools/bin/sqlcmd /usr/bin/sqlcmd  \
	&& ln -sfn /opt/mssql-tools/bin/bcp /usr/bin/bcp

RUN set -eux; \
\
	   echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
	&& locale-gen

RUN set -eux \
\
	# update pecl channel definitions https://github.com/docker-library/php/issues/443
	&& pecl update-channels \
	&& rm -rf /tmp/pear ~/.pearrc \
	&& pecl install sqlsrv \
	&& printf "; configuration for php sqlsrv module\n; priority=20\nextension=sqlsrv.so" > ${PHP_INI_DIR}/mods-available/sqlsrv.ini \
	&& pecl install pdo_sqlsrv \
	&& printf "; configuration for php pdo_sqlsrv module\n; priority=20\nextension=pdo_sqlsrv.so" > ${PHP_INI_DIR}/mods-available/pdo_sqlsrv.ini \
	&& pecl install redis \
	&& printf "; configuration for php redis module\n; priority=20\nextension=redis.so" > ${PHP_INI_DIR}/mods-available/redis.ini \
	&& pecl install xdebug \
	&& printf "; configuration for php xdebug module\n; priority=20\nzend_extension=xdebug.so" > ${PHP_INI_DIR}/mods-available/xdebug.ini \
	&& pecl install yaml \
	&& printf "; configuration for php yaml module\n; priority=20\nextension=yaml.so" > ${PHP_INI_DIR}/mods-available/xdebug.ini 

RUN set -eux; \
	\ 
	rm -rf /var/lib/apt/lists/*; 



CMD ["/bin/bash"]
