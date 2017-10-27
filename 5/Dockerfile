FROM oondeo/php:5


ENV SUMMARY="Nginx image with standar modules"	\
    DESCRIPTION="The image use scripts and configurations compatible \
        with redhat openshift."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Nginx" \
      io.openshift.s2i.scripts-url=image:///usr/libexec/s2i \
      io.s2i.scripts-url=image:///usr/libexec/s2i \
      com.redhat.component="nginx" \
      name="oondeo/php-5-nginx" \
      version="13" \
      release="1" \
      maintainer="OONDEO <info@oondeo.es>"

# Docker Build Arguments
ARG RESTY_CONFIG_OPTIONS=${RESTY_CONFIG_OPTIONS:-"\
    --with-file-aio \
    --with-http_addition_module \
    --with-http_auth_request_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_geoip_module=dynamic \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_image_filter_module=dynamic \
    --with-http_mp4_module \
    --with-http_random_index_module \
    --with-http_realip_module \
    --with-http_secure_link_module \
    --with-http_slice_module \
    --with-http_ssl_module \
    --with-http_stub_status_module \
    --with-http_sub_module \
    --with-http_v2_module \
    --with-http_xslt_module=dynamic \
    --with-ipv6 \
    --with-mail \
    --with-mail_ssl_module \
    --with-md5-asm \
    --with-pcre-jit \
    --with-sha1-asm \
    --with-stream \
    --with-stream_ssl_module \
    --with-threads \
    "
ARG RESTY_CONFIG_OPTIONS_MORE=${RESTY_CONFIG_OPTIONS_MORE:-""}

# 1) Install apt dependencies
# 2) Download and untar OpenSSL, PCRE, and OpenResty
# 3) Build OpenResty
# 4) Cleanup
USER root 

ADD s2i/bin $STI_SCRIPTS_PATH

RUN $STI_SCRIPTS_PATH/build \
    && mv /usr/local/openresty/nginx /opt/app-root/ \
    && ln -sf /opt/app-root/nginx /usr/local/openresty/nginx

# Add additional binaries into PATH for convenience
ENV PATH=$PATH:/usr/local/openresty/luajit/bin/:/usr/local/openresty/nginx/sbin/:/usr/local/openresty/bin/


ENV NGINX_CACHE=0

ADD nginx/common /opt/app-root/nginx/conf

EXPOSE 8080 8081 9000

# Reset permissions of modified directories and add default user
RUN useradd -u 1001 -r -g 0 -d ${HOME} -s /sbin/nologin \
      -c "Default Application User" default || true
RUN chown -R 1001:0 /opt/app-root \
    && chmod -R g+w /opt/app-root \
    && chmod ug+s /opt/app-root/* \
    && mkdir -p ${HOME}/.pki/nssdb \
    && chown -R 1001:0 ${HOME}/.pki

USER 1001 