FROM ejabberd/mix as builder
ARG VERSION
ENV VERSION=${VERSION:-latest} \
    MIX_ENV=prod
LABEL maintainer="ProcessOne <contact@process-one.net>" \
      product="Ejabberd Community Server builder"

# Get ejabberd sources, dependencies, configuration
RUN git clone https://github.com/processone/ejabberd.git
WORKDIR /ejabberd
COPY vars.config .
COPY rel/*exs rel/
RUN git checkout ${VERSION/latest/HEAD} \
 && mix deps.get

# Compile
RUN mix do compile, release.init, release --env=prod

# Prepare runtime environment
RUN mkdir runtime \
 && tar -C runtime -zxf _build/prod/rel/ejabberd/releases/*/ejabberd.tar.gz \
 && cd runtime \
 && cp releases/*/start.boot bin \
 && echo 'beam_lib:strip_files(filelib:wildcard("lib/*/ebin/*beam")), init:stop().' | erts*/bin/erl -boot start_clean >/dev/null \
 && mv erts*/bin/* bin \
 && rm -rf releases erts* bin/*src bin/dialyzer bin/typer \
 && rm bin/ejabberd bin/ejabberd.bat bin/ejabberd_loader.sh

# Runtime container
FROM alpine:3.8
ARG VERSION
ENV TERM=xterm \
    LC_ALL=C.UTF-8 \
    LANG=en_US.UTF-8 \
    LANGUAGE=en_US.UTF-8 \
    REPLACE_OS_VARS=true \
    HOME=/home/ejabberd \
    VERSION=${VERSION:-latest}
LABEL maintainer="ProcessOne <contact@process-one.net>" \
      product="Ejabberd Community Server" \
      version=$VERSION

# Create directory structure and user for ejabberd
RUN addgroup ejabberd -g 9000 \
 && adduser -s /bin/sh -D -G ejabberd ejabberd -u 9000 \
 && mkdir -p /home/ejabberd/conf /home/ejabberd/database /home/ejabberd/logs \
 && chown -R ejabberd:ejabberd /home/ejabberd

# Install required dependencies
RUN apk upgrade --update musl \
 && apk add \
    expat \
    gd \
    jpeg \
    libpng \
    libstdc++ \
    libwebp \
    ncurses-libs \
    openssl \
    sqlite \
    unixodbc \
    yaml \
    zlib \
 && rm -rf /var/cache/apk/*

# Install ejabberd
WORKDIR $HOME
COPY --from=builder /ejabberd/runtime .
COPY bin/* bin/
COPY --chown=ejabberd:ejabberd conf conf/
ADD --chown=ejabberd:ejabberd https://download.process-one.net/cacert.pem conf/cacert.pem

# Setup runtime environment
USER ejabberd
VOLUME ["$HOME/database","$HOME/conf","$HOME/logs"]
EXPOSE 5222 5269 5280

ENTRYPOINT ["/home/ejabberd/bin/ejabberdctl"]
CMD ["foreground"]
