ARG ALPINE_TAG=3.18
ARG CHECKOUT_TAG=v25.0
ARG BUILD_NUM_PROCS=${BUILD_NUM_PROCS:1}

###############
# Build stage #
###############
FROM alpine:${ALPINE_TAG} AS build

# Install dev dependencies
RUN apk update && \
    apk add --no-cache --update \
        build-base \
        autoconf \
        automake \
        libtool \
        pkgconfig \
        openssl-dev \
        libevent-dev \
        zeromq-dev \
        boost-dev \
        git

# Checkout source code
WORKDIR /src
ARG CHECKOUT_TAG
RUN git clone --branch ${CHECKOUT_TAG} https://github.com/bitcoin/bitcoin.git

# Build application
WORKDIR /src/bitcoin
ARG BUILD_NUM_PROCS
RUN ./autogen.sh && \
    ./configure --with-incompatible-bdb --disable-wallet --without-gui && \
    make -j${BUILD_NUM_PROCS} && \
    make install

#################
# Runtime stage #
#################
ARG ALPINE_TAG
FROM alpine:${ALPINE_TAG}

# Install runtime dependencies
RUN apk update && \
    apk add --no-cache \
    openssl \
    libevent \
    boost-filesystem \
    boost-system \
    boost-thread \
    boost-chrono \
    boost-program_options \
    libzmq \
    su-exec

# Copy executable from build to runtime
COPY --from=build /usr/local/bin/bitcoind /usr/local/bin/bitcoin-cli /usr/local/bin/

# Set up user and data directory
RUN adduser -D -u 1000 bitcoin && \
    mkdir -p /data && \
    chown -R bitcoin:bitcoin /data

# Prepare for running
EXPOSE 8333 8332
VOLUME /data
USER bitcoin

# Symlink to allow easy bitcoin-cli via docker exec
RUN ln -s /data /home/bitcoin/.bitcoin

# Start application
ENTRYPOINT ["bitcoind", \
            "-datadir=/data"]
