ARG ENABLE_PROXY=false

FROM openresty/openresty:1.19.3.1-alpine-fat AS production-stage

ARG ENABLE_PROXY
RUN set -x \
    && (test "${ENABLE_PROXY}" != "true" || /bin/sed -i 's,http://dl-cdn.alpinelinux.org,https://mirrors.aliyun.com,g' /etc/apk/repositories) \
    && apk add --no-cache --virtual .builddeps \
    automake \
    autoconf \
    libtool \
    pkgconfig \
    cmake \
    git \
    && mkdir ~/.luarocks \
    && luarocks config variables.OPENSSL_LIBDIR /usr/local/openresty/openssl/lib \
    && luarocks config variables.OPENSSL_INCDIR /usr/local/openresty/openssl/include \
    && (i=0; while true; do if [[ "$i" -eq 10 ]]; then echo "failed to fetch Lua rocks in time"; exit 1; fi; luarocks install https://github.com/apache/apisix/raw/master/rockspec/apisix-master-0.rockspec --tree=/usr/local/apisix/deps && break; i=$(( i + 1 )); done;) \
    && cp -v /usr/local/apisix/deps/lib/luarocks/rocks-5.1/apisix/master-0/bin/apisix /usr/bin/ \
    && mv /usr/local/apisix/deps/share/lua/5.1/apisix /usr/local/apisix \
    && apk del .builddeps build-base make unzip

FROM alpine:3.11 AS last-stage

ARG ENABLE_PROXY
# add runtime for Apache APISIX
RUN set -x \
    && (test "${ENABLE_PROXY}" != "true" || /bin/sed -i 's,http://dl-cdn.alpinelinux.org,https://mirrors.aliyun.com,g' /etc/apk/repositories) \
    && apk add --no-cache bash libstdc++ curl tzdata

WORKDIR /usr/local/apisix

COPY --from=production-stage /usr/local/openresty/ /usr/local/openresty/
COPY --from=production-stage /usr/local/apisix/ /usr/local/apisix/
COPY --from=production-stage /usr/bin/apisix /usr/bin/apisix

ENV PATH=$PATH:/usr/local/openresty/luajit/bin:/usr/local/openresty/nginx/sbin:/usr/local/openresty/bin

EXPOSE 9080 9443

CMD ["sh", "-c", "/usr/bin/apisix init && /usr/bin/apisix init_etcd && /usr/local/openresty/bin/openresty -p /usr/local/apisix -g 'daemon off;'"]

STOPSIGNAL SIGQUIT
