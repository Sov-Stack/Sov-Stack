###############
# Build stage #
###############
FROM alpine:latest AS build

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
RUN git clone https://git.torproject.org/tor.git && \
    cd tor && \
    git checkout tor-0.4.7.13

# Build application
WORKDIR /src/tor
RUN ./autogen.sh && \
    ./configure --disable-asciidoc && \
    make

#################
# Runtime stage #
#################
FROM alpine:latest

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