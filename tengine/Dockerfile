FROM alpine:latest


ENV TENGINE_VERSION 3.1.0

ENV CONFIG "\
        --with-http_v2_module \
		--with-http_realip_module \
		--with-http_gzip_static_module \
		--with-http_mp4_module \
		--add-module=./modules/ngx_http_trim_filter_module \
		--with-openssl=/root/openssl-3.4.0  \
		--with-openssl-opt=enable-tls1_3 \
		--add-module=/root/ngx_devel_kit-0.3.3 \
		--add-module=/root/lua-nginx-module-0.10.27 \
		--with-luajit-inc=/usr/local/include/luajit-2.1 \
		--with-luajit-lib=/usr/local/lib \
		--with-ld-opt=-Wl,-rpath,/usr/local/lib \
        "

WORKDIR /root
COPY . .
RUN apk add --no-cache make gcc musl-dev linux-headers perl pcre-dev openssl-dev zlib-dev \
	&& cd luajit2-2.1-20240815 \
	&& make -j "$(nproc)" \
	&& make install \
	&& cd ../tengine-$TENGINE_VERSION \
	&& ./configure $CONFIG \
	&& make -j "$(nproc)" \
	&& make install \
	&& cd ../lua-resty-core-0.1.30 \
	&& make install PREFIX=/usr/local/nginx \	
	&& cd ../lua-resty-lrucache-0.15 \
	&& make install PREFIX=/usr/local/nginx \
	&& addgroup -S www \
	&& adduser -D -S -h /home -s /sbin/nologin -G www www \
	&& ln -s /usr/local/nginx/sbin/nginx /usr/local/bin/nginx \
	&& adduser -u 82 -D -S -G www-data www-data \
	&& cd /root \
	&& \cp -f nginx.conf /usr/local/nginx/conf \
	&& apk del gcc linux-headers musl-dev make perl openssl-dev \
	&& rm -rf ~/* ~/.git ~/.gitignore ~/.travis.yml ~/.ash_history \
	&& rm -rf /root/* \
	&& rm -rf /var/cache/apk/*

EXPOSE 80 443

STOPSIGNAL SIGTERM

CMD ["nginx", "-g", "daemon off;"]
