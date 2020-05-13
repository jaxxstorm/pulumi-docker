# syntax = docker/dockerfile:experimental
# Interim container so we can copy pulumi binaries
# Must be defined first
ARG PULUMI_VERSION=latest
FROM jaxxstorm/pulumi:base.${PULUMI_VERSION} as pulumi

# Build container
FROM ubuntu:bionic AS builder

ARG GO_VERSION=1.13.9
ARG GO_SHA256SUM=8a4cbc9f2b95d114c38f6cbe94a45372d48c604b707db2057c787398dfbf8e7f
WORKDIR /golang
RUN --mount=target=/var/lib/apt/lists,type=cache \
    --mount=target=/var/cache/apt,type=cache \
    apt-get update -y && \
      apt-get install -y \
      curl \
      git \
      build-essential \
      git
RUN curl -fsSLo /tmp/go.tgz https://golang.org/dl/go${GO_VERSION}.linux-amd64.tar.gz; \
    echo "${GO_SHA256SUM} /tmp/go.tgz" | sha256sum -c -; \
    mkdir -p bin; \
    tar -C /golang -xzf /tmp/go.tgz; \
    rm /tmp/go.tgz; \
    export PATH="/golang/go/bin:${PATH}"; \
    go version

# The runtime container
# This is our base container, so let's copy all the runtimes to .pulumi/bin
FROM ubuntu:bionic
WORKDIR /pulumi/projects
ARG PULUMI_VERSION=latest

# Install needed tools, like git
RUN --mount=target=/var/lib/apt/lists,type=cache \
    --mount=target=/var/cache/apt,type=cache \
    apt-get update -y && \
    apt-get install -y \
    git; \
    mkdir -p /go

# Uses the workdir
COPY --from=builder /golang/go /usr/local
# Uses the workdir, copies from pulumi interim container
COPY --from=pulumi /pulumi/bin/pulumi /pulumi/bin/pulum
COPY --from=pulumi /pulumi/bin/pulumi-language-go /pulumi/bin/pulumi-language-go
COPY --from=pulumi /pulumi/bin/pulumi-analyzer-policy /pulumi/bin/pulumi-language-go
ENV GOPATH=/go
ENV PATH "/pulumi/bin:${PATH}"

ENTRYPOINT ["pulumi"]
