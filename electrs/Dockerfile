###############
# Build stage #
###############
FROM alpine:latest AS build

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
RUN git clone --branch v0.9.13 https://github.com/romanz/electrs.git

# Build application
WORKDIR /src/electrs
RUN cargo build --release

#################
# Runtime stage #
#################
FROM alpine:latest

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