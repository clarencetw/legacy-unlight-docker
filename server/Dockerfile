ARG APP_ROOT=/app
ARG RUBY_VERSION=2.6.5

# Pre-compile Gems
FROM ruby:${RUBY_VERSION}-alpine AS gem

RUN apk --update \
        add --no-cache \
        build-base \
        ca-certificates \
        zlib-dev \
        libressl-dev \
        mariadb-dev \
        mariadb-client \
        sqlite \
        sqlite-dev

# Setup Application
ARG APP_ROOT
ENV APP_ROOT=${APP_ROOT}
ENV SERVER_ROOT=$APP_ROOT/server

RUN mkdir -p $SERVER_ROOT
WORKDIR $SERVER_ROOT

COPY server/Gemfile* $SERVER_ROOT/
RUN cd $SERVER_ROOT \
    && gem install bundler:1.17.3 \
    && gem uninstall bundler:1.17.2 \
    && bundle config --global deployment 'true' \
    && bundle config --global frozen 'true' \
    && bundle config --global without 'development test' \
    && bundle install --no-cache \
    && find /usr/local/bundle -type f -name '*.c' -delete \
    && find /usr/local/bundle -type f -name '*.o' -delete \
    && rm -rf /usr/local/bundle/cache/*.gem

# Server
FROM ruby:${RUBY_VERSION}-alpine

RUN apk --update \
        add --no-cache \
        gcc \
        libc-dev \
        ca-certificates \
        zlib-dev \
        libressl-dev \
        mariadb-connector-c \
        mariadb-client \
        sqlite

ARG APP_ROOT
ENV APP_ROOT=${APP_ROOT}
ENV SERVER_ROOT=$APP_ROOT/server

# Setup Application
RUN mkdir -p $SERVER_ROOT \
      && mkdir -p $SERVER_ROOT/bin/pids \
      && mkdir -p $SERVER_ROOT/data/backup
WORKDIR $SERVER_ROOT

COPY --from=gem /usr/local/bundle /usr/local/bundle

# Add Source Files
ADD src/app/server/ $SERVER_ROOT
ADD server/ $SERVER_ROOT
ADD patches/ $APP_ROOT/patches

ENV PATH $SERVER_ROOT/bin:$PATH

# Patch for Dockerize
RUN sh $SERVER_ROOT/scripts/prepare.sh \
    && sh $SERVER_ROOT/scripts/patch.sh
