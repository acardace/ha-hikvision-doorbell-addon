ARG BUILD_FROM=ghcr.io/home-assistant/amd64-base:latest

# Stage 1: Build the Go binary
FROM golang:1.26-alpine AS builder

WORKDIR /build

# Install git for fetching dependencies
RUN apk add --no-cache git

# Clone the doorbell server repository
RUN git clone --depth 1 https://github.com/acardace/hikvision-doorbell-server.git .

# Download dependencies
RUN go mod download

# Build static binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -ldflags '-extldflags "-static"' -o doorbell-server ./cmd/server

# Stage 2: Final image
FROM ${BUILD_FROM}

# Install runtime dependencies
RUN apk add --no-cache jq ffmpeg

# Copy the built binary
COPY --from=builder /build/doorbell-server /usr/bin/doorbell-server

# Copy integration (custom_components)
COPY integration/custom_components /custom_components

# Copy card JS into integration's www folder for auto-registration
RUN mkdir -p /custom_components/hikvision_doorbell/www
COPY card/hikvision-doorbell-card.js /custom_components/hikvision_doorbell/www/

# Copy run script
COPY run.sh /run.sh
RUN chmod a+x /run.sh

CMD ["/run.sh"]
