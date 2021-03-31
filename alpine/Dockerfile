FROM alpine:3.13

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN addgroup -g 11211 memcache && adduser -D -u 11211 -G memcache memcache

# ensure SASL's "libplain.so" is installed as per https://github.com/memcached/memcached/wiki/SASLHowto
RUN apk add --no-cache libsasl
RUN set -x \
	\
	&& apk add --no-cache --virtual .build-deps \
		ca-certificates \
		coreutils \
		cyrus-sasl-dev \
		gcc \
		libc-dev \
		libevent-dev \
		linux-headers \
		make \
		openssl \
		openssl-dev \
		perl \
		perl-io-socket-ssl \
		perl-utils 

ADD memcached-1.6.9/ /usr/src/memcached
	
RUN cd /usr/src/memcached \
	\
	&& ./configure \
		--build="$gnuArch" \
		--enable-extstore \
		--enable-sasl \
		--enable-sasl-pwdb \
		--enable-tls \
	&& nproc="$(nproc)" \
	&& make -j "$nproc" \
	\
	&& make test PARALLEL="$nproc" \
	\
	&& make install \
	\
	&& cd / && rm -rf /usr/src/memcached \
	\
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-network --virtual .memcached-rundeps $runDeps \
	&& apk del --no-network .build-deps \
	\
	&& memcached -V

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

USER memcache
EXPOSE 11211
CMD ["memcached"]
