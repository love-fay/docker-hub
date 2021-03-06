FROM debian:buster-slim

RUN groupadd --gid 1000 node \
  && useradd --uid 1000 --gid node --shell /bin/bash --create-home node

ENV NODE_VERSION 10.16.3

RUN buildDeps='xz-utils' \
    && ARCH= && dpkgArch="$(dpkg --print-architecture)" \
    && case "${dpkgArch##*-}" in \
      amd64) ARCH='x64';; \
      ppc64el) ARCH='ppc64le';; \
      s390x) ARCH='s390x';; \
      arm64) ARCH='arm64';; \
      armhf) ARCH='armv7l';; \
      i386) ARCH='x86';; \
      *) echo "unsupported architecture"; exit 1 ;; \
    esac \
    && set -ex \
    && apt-get update && apt-get install -y ca-certificates curl wget gnupg dirmngr $buildDeps --no-install-recommends \
    && rm -rf /var/lib/apt/lists/* \
    && for key in \
      94AE36675C464D64BAFA68DD7434390BDBE9B9C5 \
      FD3A5288F042B6850C66B31F09FE44734EB7990E \
      71DCFD284A79C3B38668286BC97EC7A07EDE3FC1 \
      DD8F2338BAE7501E3DD5AC78C273792F7D83545D \
      C4F0DFFF4E8C1A8236409D08E73BC641CC11F4C8 \
      B9AE9905FFD7803F25714661B63B535A4C206CA9 \
      77984A986EBC2AA786BC0F66B01FBB92821C587A \
      8FCCA13FEF1D0C2E91008E09770F7A9A5AE15600 \
      4ED778F539E3634C779C87C6D7062848A1AB005C \
      A48C2BEE680E841632CD4E44F07496B3EB3C1762 \
      B9E2F5981AA6E0CD28160D9FF13993A75599653C \
    ; do \
      gpg --batch --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys "$key" || \
      gpg --batch --keyserver hkp://ipv4.pool.sks-keyservers.net --recv-keys "$key" || \
      gpg --batch --keyserver hkp://pgp.mit.edu:80 --recv-keys "$key" ; \
    done \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-$ARCH.tar.xz" \
    && curl -fsSLO --compressed "https://nodejs.org/dist/v$NODE_VERSION/SHASUMS256.txt.asc" \
    && gpg --batch --decrypt --output SHASUMS256.txt SHASUMS256.txt.asc \
    && grep " node-v$NODE_VERSION-linux-$ARCH.tar.xz\$" SHASUMS256.txt | sha256sum -c - \
    && tar -xJf "node-v$NODE_VERSION-linux-$ARCH.tar.xz" -C /usr/local --strip-components=1 --no-same-owner \
    && rm "node-v$NODE_VERSION-linux-$ARCH.tar.xz" SHASUMS256.txt.asc SHASUMS256.txt \
    && apt-get purge -y --auto-remove $buildDeps \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs

#ENV PATH /usr/local/bin:$PATH
#
## http://bugs.python.org/issue19846
## > At the moment, setting "LANG=C" on a Linux system *fundamentally breaks Python 3*, and that's not OK.
#ENV LANG C.UTF-8
#
## runtime dependencies
#RUN apt-get update && apt-get install -y --no-install-recommends \
#		ca-certificates \
#		netbase \
#	&& rm -rf /var/lib/apt/lists/*
#
#ENV GPG_KEY 0D96DF4D4110E5C43FBFB17F2D347EA6AA65421D
#ENV PYTHON_VERSION 3.7.4
#
#RUN set -ex \
#	\
#	&& savedAptMark="$(apt-mark showmanual)" \
#	&& apt-get update && apt-get install -y --no-install-recommends \
#		dpkg-dev \
#		gcc \
#		libbz2-dev \
#		libc6-dev \
#		libexpat1-dev \
#		libffi-dev \
#		libgdbm-dev \
#		liblzma-dev \
#		libncursesw5-dev \
#		libreadline-dev \
#		libsqlite3-dev \
#		libssl-dev \
#		make \
#		tk-dev \
#		uuid-dev \
#		wget \
#		xz-utils \
#		zlib1g-dev \
## as of Stretch, "gpg" is no longer included by default
#		$(command -v gpg > /dev/null || echo 'gnupg dirmngr') \
#	\
#	&& wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
#	&& wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
#	&& export GNUPGHOME="$(mktemp -d)" \
#	&& gpg --batch --keyserver ha.pool.sks-keyservers.net --recv-keys "$GPG_KEY" \
#	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
#	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
#	&& rm -rf "$GNUPGHOME" python.tar.xz.asc \
#	&& mkdir -p /usr/src/python \
#	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
#	&& rm python.tar.xz \
#	\
#	&& cd /usr/src/python \
#	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
#	&& ./configure \
#		--build="$gnuArch" \
#		--enable-loadable-sqlite-extensions \
#		--enable-optimizations \
#		--enable-shared \
#		--with-system-expat \
#		--with-system-ffi \
#		--without-ensurepip \
#	&& make -j "$(nproc)" \
## setting PROFILE_TASK makes "--enable-optimizations" reasonable: https://bugs.python.org/issue36044 / https://github.com/docker-library/python/issues/160#issuecomment-509426916
#		PROFILE_TASK='-m test.regrtest --pgo \
#			test_array \
#			test_base64 \
#			test_binascii \
#			test_binhex \
#			test_binop \
#			test_bytes \
#			test_c_locale_coercion \
#			test_class \
#			test_cmath \
#			test_codecs \
#			test_compile \
#			test_complex \
#			test_csv \
#			test_decimal \
#			test_dict \
#			test_float \
#			test_fstring \
#			test_hashlib \
#			test_io \
#			test_iter \
#			test_json \
#			test_long \
#			test_math \
#			test_memoryview \
#			test_pickle \
#			test_re \
#			test_set \
#			test_slice \
#			test_struct \
#			test_threading \
#			test_time \
#			test_traceback \
#			test_unicode \
#		' \
#	&& make install \
#	&& ldconfig \
#	\
#	&& apt-mark auto '.*' > /dev/null \
#	&& apt-mark manual $savedAptMark \
#	&& find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec ldd '{}' ';' \
#		| awk '/=>/ { print $(NF-1) }' \
#		| sort -u \
#		| xargs -r dpkg-query --search \
#		| cut -d: -f1 \
#		| sort -u \
#		| xargs -r apt-mark manual \
#	&& apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
#	&& rm -rf /var/lib/apt/lists/* \
#	\
#	&& find /usr/local -depth \
#		\( \
#			\( -type d -a \( -name test -o -name tests \) \) \
#			-o \
#			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
#		\) -exec rm -rf '{}' + \
#	&& rm -rf /usr/src/python \
#	\
#	&& python3 --version
#
## make some useful symlinks that are expected to exist
#RUN cd /usr/local/bin \
#	&& ln -s idle3 idle \
#	&& ln -s pydoc3 pydoc \
#	&& ln -s python3 python \
#	&& ln -s python3-config python-config
#
## if this is called "PIP_VERSION", pip explodes with "ValueError: invalid truth value '<VERSION>'"
#ENV PYTHON_PIP_VERSION 19.2.2
## https://github.com/pypa/get-pip
#ENV PYTHON_GET_PIP_URL https://github.com/pypa/get-pip/raw/0c72a3b4ece313faccb446a96c84770ccedc5ec5/get-pip.py
#ENV PYTHON_GET_PIP_SHA256 201edc6df416da971e64cc94992d2dd24bc328bada7444f0c4f2031ae31e8dad
#
#RUN set -ex; \
#	\
#	savedAptMark="$(apt-mark showmanual)"; \
#	apt-get update; \
#	apt-get install -y --no-install-recommends wget; \
#	\
#	wget -O get-pip.py "$PYTHON_GET_PIP_URL"; \
#	echo "$PYTHON_GET_PIP_SHA256 *get-pip.py" | sha256sum --check --strict -; \
#	\
#	apt-mark auto '.*' > /dev/null; \
#	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
#	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
#	rm -rf /var/lib/apt/lists/*; \
#	\
#	python get-pip.py \
#		--disable-pip-version-check \
#		--no-cache-dir \
#		"pip==$PYTHON_PIP_VERSION" \
#	; \
#	pip --version; \
#	\
#	find /usr/local -depth \
#		\( \
#			\( -type d -a \( -name test -o -name tests \) \) \
#			-o \
#			\( -type f -a \( -name '*.pyc' -o -name '*.pyo' \) \) \
#		\) -exec rm -rf '{}' +; \
#	rm -f get-pip.py

ENV NGINX_VERSION   1.17.3
ENV NJS_VERSION     0.3.5
ENV PKG_RELEASE     1~buster

RUN set -x \
# create nginx user/group first, to be consistent throughout docker variants
    && addgroup --system --gid 101 nginx \
    && adduser --system --disabled-login --ingroup nginx --no-create-home --home /nonexistent --gecos "nginx user" --shell /bin/false --uid 101 nginx \
    && apt-get update \
    && apt-get install --no-install-recommends --no-install-suggests -y gnupg1 ca-certificates \
    && \
    NGINX_GPGKEY=573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62; \
    found=''; \
    for server in \
        ha.pool.sks-keyservers.net \
        hkp://keyserver.ubuntu.com:80 \
        hkp://p80.pool.sks-keyservers.net:80 \
        pgp.mit.edu \
    ; do \
        echo "Fetching GPG key $NGINX_GPGKEY from $server"; \
        apt-key adv --keyserver "$server" --keyserver-options timeout=10 --recv-keys "$NGINX_GPGKEY" && found=yes && break; \
    done; \
    test -z "$found" && echo >&2 "error: failed to fetch GPG key $NGINX_GPGKEY" && exit 1; \
    apt-get remove --purge --auto-remove -y gnupg1 && rm -rf /var/lib/apt/lists/* \
    && dpkgArch="$(dpkg --print-architecture)" \
    && nginxPackages=" \
        nginx=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-xslt=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-geoip=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-image-filter=${NGINX_VERSION}-${PKG_RELEASE} \
        nginx-module-njs=${NGINX_VERSION}.${NJS_VERSION}-${PKG_RELEASE} \
    " \
    && case "$dpkgArch" in \
        amd64|i386) \
# arches officialy built by upstream
            echo "deb https://nginx.org/packages/mainline/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list \
            && apt-get update \
            ;; \
        *) \
# we're on an architecture upstream doesn't officially build for
# let's build binaries from the published source packages
            echo "deb-src https://nginx.org/packages/mainline/debian/ buster nginx" >> /etc/apt/sources.list.d/nginx.list \
            \
# new directory for storing sources and .deb files
            && tempDir="$(mktemp -d)" \
            && chmod 777 "$tempDir" \
# (777 to ensure APT's "_apt" user can access it too)
            \
# save list of currently-installed packages so build dependencies can be cleanly removed later
            && savedAptMark="$(apt-mark showmanual)" \
            \
# build .deb files from upstream's source packages (which are verified by apt-get)
            && apt-get update \
            && apt-get build-dep -y $nginxPackages \
            && ( \
                cd "$tempDir" \
                && DEB_BUILD_OPTIONS="nocheck parallel=$(nproc)" \
                    apt-get source --compile $nginxPackages \
            ) \
# we don't remove APT lists here because they get re-downloaded and removed later
            \
# reset apt-mark's "manual" list so that "purge --auto-remove" will remove all build dependencies
# (which is done after we install the built packages so we don't have to redownload any overlapping dependencies)
            && apt-mark showmanual | xargs apt-mark auto > /dev/null \
            && { [ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; } \
            \
# create a temporary local APT repo to install from (so that dependency resolution can be handled by APT, as it should be)
            && ls -lAFh "$tempDir" \
            && ( cd "$tempDir" && dpkg-scanpackages . > Packages ) \
            && grep '^Package: ' "$tempDir/Packages" \
            && echo "deb [ trusted=yes ] file://$tempDir ./" > /etc/apt/sources.list.d/temp.list \
# work around the following APT issue by using "Acquire::GzipIndexes=false" (overriding "/etc/apt/apt.conf.d/docker-gzip-indexes")
#   Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
#   ...
#   E: Failed to fetch store:/var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages  Could not open file /var/lib/apt/lists/partial/_tmp_tmp.ODWljpQfkE_._Packages - open (13: Permission denied)
            && apt-get -o Acquire::GzipIndexes=false update \
            ;; \
    esac \
    \
    && apt-get install --no-install-recommends --no-install-suggests -y \
                        $nginxPackages \
                        gettext-base \
    && apt-get remove --purge --auto-remove -y ca-certificates && rm -rf /var/lib/apt/lists/* /etc/apt/sources.list.d/nginx.list \
    \
# if we have leftovers from building, let's purge them (including extra, unnecessary build deps)
    && if [ -n "$tempDir" ]; then \
        apt-get purge -y --auto-remove \
        && rm -rf "$tempDir" /etc/apt/sources.list.d/temp.list; \
    fi

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log \
    && ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80

STOPSIGNAL SIGTERM


CMD ["nginx", "-g", "daemon off;"]