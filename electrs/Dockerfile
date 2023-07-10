ARG ALPINE_TAG=3.18
ARG CHECKOUT_TAG=v0.9.13
ARG BUILD_NUM_PROCS=${BUILD_NUM_PROCS:1}

###############
# Build stage #
###############
FROM alpine:${ALPINE_TAG} AS build

# Install dev dependencies
RUN apk update && \
    apk add --no-cache --update \
        build-base \
        linux-headers \
        git \
        clang15 \
        cmake \
        pkgconfig \
        rust \
        cargo \
        libressl-dev

# Checkout source code
WORKDIR /src
ARG CHECKOUT_TAG
RUN git clone --branch ${CHECKOUT_TAG} https://github.com/romanz/electrs.git

# Build application
WORKDIR /src/electrs
ARG BUILD_NUM_PROCS
ENV CARGO_BUILD_JOBS=${BUILD_NUM_PROCS}
RUN cargo build --release

#################
# Runtime stage #
#################
ARG ALPINE_TAG
FROM alpine:${ALPINE_TAG}

# Install runtime dependencies
RUN apk update && \
    apk add --no-cache \
        llvm16-libs \
        libressl-dev

# Copy executable from build to runtime
COPY --from=build /src/electrs/target/release/electrs /usr/local/bin/

# Set up user and Bitcoin data directory
RUN adduser -D -u 1000 electrs && \
    mkdir -p /bitcoin /data && \
    chown -R electrs:electrs /bitcoin && \
    chown -R electrs:electrs /data

# Prepare for running
EXPOSE 50001
VOLUME /bitcoin
VOLUME /data
USER electrs

# Start application
ENTRYPOINT [ "electrs" ]
