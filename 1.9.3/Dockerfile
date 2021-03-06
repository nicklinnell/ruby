FROM alpine:3.4

# skip installing gem documentation
RUN mkdir -p /usr/local/etc \
      && { \
        echo 'install: --no-document'; \
          echo 'update: --no-document'; \
      } >> /usr/local/etc/gemrc

ENV RUBY_MAJOR 1.9
ENV RUBY_VERSION 1.9.3-p551
ENV RUBY_DOWNLOAD_SHA256 bb5be55cd1f49c95bb05b6f587701376b53d310eb1bb7c76fbd445a1c75b51e8
ENV RUBYGEMS_VERSION 1.8.23.2

# some of ruby's build scripts are written in ruby
# we purge this later to make sure our final image uses what we just built
RUN set -ex \
      && apk add --no-cache --virtual .ruby-builddeps \
        autoconf \
        bison \
        bzip2 \
        bzip2-dev \
        ca-certificates \
        coreutils \
        curl \
        gcc \
        gdbm-dev \
        glib-dev \
        libc-dev \
        libffi-dev \
        libxml2-dev \
        libxslt-dev \
        linux-headers \
        make \
        ncurses-dev \
        openssl-dev \
        procps \
# https://bugs.ruby-lang.org/issues/11869 and https://github.com/docker-library/ruby/issues/75
        readline-dev \
        ruby \
        yaml-dev \
        zlib-dev \
      && curl -fSL -o ruby.tar.gz "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.gz" \
      && echo "$RUBY_DOWNLOAD_SHA256 *ruby.tar.gz" | sha256sum -c - \
      && mkdir -p /usr/src \
      && tar -xzf ruby.tar.gz -C /usr/src \
      && mv "/usr/src/ruby-$RUBY_VERSION" /usr/src/ruby \
      && rm ruby.tar.gz \
      && cd /usr/src/ruby \
      && { echo '#define ENABLE_PATH_CHECK 0'; echo; cat file.c; } > file.c.new && mv file.c.new file.c \
      && autoconf \
# the configure script does not detect isnan/isinf as macros
      && ac_cv_func_isnan=yes ac_cv_func_isinf=yes \
      ./configure --disable-install-doc \
      && make -j"$(getconf _NPROCESSORS_ONLN)" \
      && make install \
      && runDeps="$( \
      scanelf --needed --nobanner --recursive /usr/local \
      | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
      | sort -u \
      | xargs -r apk info --installed \
      | sort -u \
      )" \
      && apk add --virtual .ruby-rundeps $runDeps \
        bzip2 \
        ca-certificates \
        curl \
        libffi-dev \
        openssl-dev \
        yaml-dev \
        procps \
        zlib-dev \
      && apk del .ruby-builddeps \
      && gem update --system $RUBYGEMS_VERSION \
      && rm -r /usr/src/ruby

ENV BUNDLER_VERSION %%BUNDLER%%

RUN gem install bundler --version "$BUNDLER_VERSION"

# install things globally, for great justice
# and don't create ".bundle" in all our apps
ENV GEM_HOME /usr/local/bundle
ENV BUNDLE_PATH="$GEM_HOME" \
     BUNDLE_BIN="$GEM_HOME/bin" \
     BUNDLE_SILENCE_ROOT_WARNING=1 \
BUNDLE_APP_CONFIG="$GEM_HOME"
ENV PATH $BUNDLE_BIN:$PATH
RUN mkdir -p "$GEM_HOME" "$BUNDLE_BIN" \
     && chmod 777 "$GEM_HOME" "$BUNDLE_BIN"

CMD [ "irb" ]
