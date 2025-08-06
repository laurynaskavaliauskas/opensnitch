# Standalone Docker file for builds without requiring any files pulled from lidlabs. It is meant for testing and will not be updated regularly.
FROM golang:1.20.14-bookworm  as builder

WORKDIR /app

#Install required Go libraries

RUN go install github.com/golang/protobuf/protoc-gen-go@latest && \
    go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.34.1 && \
    go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.3.0

# Apt optimized example taken from https://github.com/cogini/phoenix_container_example/blob/main/deploy/debian.Dockerfile#L113
RUN --mount=type=cache,id=apt-cache,target=/var/cache/apt,sharing=locked \
--mount=type=cache,id=apt-lib,target=/var/lib/apt,sharing=locked \
--mount=type=cache,id=debconf,target=/var/cache/debconf,sharing=locked \
set -exu && \
apt-get update -qq && \
DEBIAN_FRONTEND=noninteractive \
apt-get install -y -qq --no-install-recommends \
    bash \
    protobuf-compiler \
    ca-certificates \
    git \
    gpg \
    make \
    python3-grpc-tools python3-pkg-resources python3-netfilter libnetfilter-queue-dev

COPY . .

RUN cd ./proto && make
RUN cd ./daemon && go mod tidy
RUN cd ./daemon && make




