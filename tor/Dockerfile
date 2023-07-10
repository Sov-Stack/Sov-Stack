ARG ALPINE_TAG=3.18
ARG CHECKOUT_TAG=tor-0.4.7.13
ARG BUILD_NUM_PROCS=${BUILD_NUM_PROCS:1}

###############
# Build stage #
###############
FROM alpine:${ALPINE_TAG} AS build

# Install dev dependencies
RUN apk update && \
    apk add --no-cache --update \
        gnupg \
        build-base \
        gcc \
        wget \
        git \
        libevent \
        libevent-dev \
        xz-libs \
        xz-dev \
        zlib \
        zlib-dev \
        zstd \
        zstd-libs \
        zstd-dev \
        automake \
        autoconf \
        openssl-dev

# Checkout source code
WORKDIR /src
ARG CHECKOUT_TAG
RUN git clone --branch ${CHECKOUT_TAG} https://git.torproject.org/tor.git

# Build application
WORKDIR /src/tor
ARG BUILD_NUM_PROCS
RUN ./autogen.sh && \
    ./configure --disable-asciidoc && \
    make -j${BUILD_NUM_PROCS}

#################
# Runtime stage #
#################
ARG ALPINE_TAG
FROM alpine:${ALPINE_TAG}

# Install runtime dependencies
RUN apk update && \
    apk add --no-cache --update \
        libevent \
        xz-libs \
        zstd-libs \
        openssl \
        zlib

# Copy executable from build to runtime
COPY --from=build /src/tor/src/app/tor /usr/local/bin

# Set up user, config and data directories
RUN adduser -D -u 1000 tor && \
    mkdir -p /tor/cfg /tor/data && \
    chown -R tor:tor /tor

# Prepare for running
EXPOSE 9050 9051
VOLUME /tor
USER tor

# Start application
ENTRYPOINT ["tor", \
            "-f", "/tor/cfg/torrc"]
