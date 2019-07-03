FROM nginx:1.17

# Install OpenSSL
RUN apt-get update && apt-get install --no-install-recommends --no-install-suggests -y openssl

# Generate an SSL certificate
RUN mkdir -p /etc/ssl/test

RUN openssl genrsa -out "/etc/ssl/test/test.key" 2048

RUN openssl req \
  -new \
  -key "/etc/ssl/test/test.key" \
  -out "/etc/ssl/test/test.csr" \
  -subj "/C=GB/ST=London/L=London/O=Local VM/CN=*.test"

RUN openssl x509 -req -days 365 \
    -in "/etc/ssl/test/test.csr" \
    -signkey "/etc/ssl/test/test.key" \
    -out "/etc/ssl/test/test.crt"

# Copy the config files
COPY ./config/nginx.conf /etc/nginx/nginx.conf
COPY ./config/fastcgi_params /etc/nginx/fastcgi_params

VOLUME ["/etc/nginx/conf.d", "/var/code"]
