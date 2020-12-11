FROM debian:buster-slim

RUN apt-get update && apt-get install -y ca-certificates curl --no-install-recommends\
    && curl https://getcaddy.com | bash -s personal && cd / && mkdir fay && cd fay && mkdir caddy

WORKDIR /fay/caddy

EXPOSE 80 2015 443

STOPSIGNAL SIGTERM


CMD ["caddy", "--log", "stdout", "-agree"]