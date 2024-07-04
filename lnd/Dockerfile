# Mostly copied from lightningnetwork/lnd/Dockerfile (which is under MIT license),
# adapted to Sov-Stack conventions like tag checkouts, credentials and file structure.

ARG ALPINE_TAG=3.18
ARG CHECKOUT_TAG=v0.17.3-beta
ARG BUILD_NUM_PROCS=${BUILD_NUM_PROCS:1}

###############
# Build stage #
###############
FROM golang:1.21.0-alpine as builder

# Force Go to use the cgo based DNS resolver. This is required to ensure DNS
# queries required to connect to linked containers succeed.
ENV GODEBUG netdns=cgo

# Install dev dependencies
RUN apk add --no-cache --update alpine-sdk \
    git \
    make \
    gcc

# Checkout source code
WORKDIR /src
ARG CHECKOUT_TAG
RUN git clone --branch ${CHECKOUT_TAG} https://github.com/lightningnetwork/lnd.git

# Build application
WORKDIR /src/lnd
ARG BUILD_NUM_PROCS
RUN make -j${BUILD_NUM_PROCS} release-install

#################
# Runtime stage #
#################
ARG ALPINE_TAG
FROM alpine:${ALPINE_TAG}

# Add runtime dependencies
RUN apk --no-cache add \
    bash \
    jq \
    ca-certificates \
    gnupg \
    curl

# Copy binaries over
COPY --from=builder /go/bin/lncli /bin/
COPY --from=builder /go/bin/lnd /bin/
COPY --from=builder /src/lnd/scripts/verify-install.sh /
COPY --from=builder /src/lnd/scripts/keys/* /keys/

# Expose P2P and RPC ports
EXPOSE 9735 10009

VOLUME /root/.lnd
VOLUME /tor

ENTRYPOINT ["lnd"]
