FROM alpine:3.19

# 1. 패키지 설치 (이 단계는 캐시로 빠르게 반복 가능)
RUN apk add --no-cache \
    bash \
    git \
    build-base \
    pcre-dev \
    zlib-dev \
    openssl-dev \
    linux-headers \
    wget \
    python3 \
    py3-pip \
    iptables \
    fail2ban \
    supervisor

# 2. 소스코드 다운 (캐시 활용)
ENV NGINX_VERSION=1.25.4

WORKDIR /build

# 2-1. naxsi 소스 다운로드
RUN git clone --recurse-submodules https://github.com/wargio/naxsi.git naxsi

# 2-2. nginx 소스 다운로드 및 압축 해제
RUN wget http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz -O nginx.tar.gz \
    && tar -xzf nginx.tar.gz

# 3. nginx + naxsi 모듈 빌드 (이 단계만 다시 빌드해도 됨)
WORKDIR /build/nginx-${NGINX_VERSION}
RUN ./configure \
      --prefix=/usr/local/nginx \
      --sbin-path=/usr/sbin/nginx \
      --conf-path=/etc/nginx/nginx.conf \
      --error-log-path=/var/log/nginx/error.log \
      --http-log-path=/var/log/nginx/access.log \
      --pid-path=/run/nginx/nginx.pid \
      --lock-path=/run/nginx/nginx.lock \
      --with-http_ssl_module \
      --with-threads \
      --add-dynamic-module=/build/naxsi/naxsi_src \
    && make \
    && make install

# 4. 나머지 설정/디렉토리 생성 및 파일 복사
WORKDIR /
RUN mkdir -p /run/nginx /etc/nginx/modules-enabled /var/log/nginx /var/www/certbot /etc/fail2ban/filter.d /var/log/fail2ban \
    && touch /var/log/fail2ban.log \
    && chmod 666 /var/log/fail2ban.log
RUN mkdir -p /etc/nginx/modules && \
    ln -s /usr/local/nginx/modules/ngx_http_naxsi_module.so /etc/nginx/modules/ngx_http_naxsi_module.so

# 5. nginx, naxsi, fail2ban, supervisor 설정 복사
COPY supervisord.conf /etc/supervisord.conf
COPY fail2ban/jail.local /etc/fail2ban/jail.local
COPY fail2ban/filter.d/nginx-scanner-block.conf /etc/fail2ban/filter.d/nginx-scanner-block.conf

# Alpine 기본 ssh jail 제거 (문제 방지)
RUN rm -f /etc/fail2ban/jail.d/alpine-ssh.conf

COPY nginx/nginx.conf /etc/nginx/nginx.conf
COPY nginx/naxsi_core.rules /etc/nginx/naxsi_core.rules
COPY nginx/naxsi_whitelist.rules /etc/nginx/naxsi_whitelist.rules
COPY nginx/naxsi.rules /etc/nginx/naxsi.rules

VOLUME ["/etc/nginx", "/var/log/nginx", "/etc/fail2ban", "/var/www/certbot", "/var/log/fail2ban"]

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
