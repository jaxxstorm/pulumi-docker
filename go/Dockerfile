# syntax = docker/dockerfile:experimental
# Interim container so we can copy pulumi binaries
# Must be defined first
ARG PULUMI_VERSION=latest
ARG PULUMI_IMAGE=jaxxstorm/pulumi
FROM ${PULUMI_IMAGE}:base.${PULUMI_VERSION} as pulumi

# Build container
FROM ubuntu:bionic AS builder

# Set go versions
ARG RUNTIME_VERSION=1.13.9

WORKDIR /golang
RUN apt-get update -y && \
      apt-get install -y \
      curl \
      git \
      build-essential \
      git
RUN curl -fsSLo /tmp/go.tgz https://golang.org/dl/go${RUNTIME_VERSION}.linux-amd64.tar.gz; \
    mkdir -p bin; \
    tar -C /golang -xzf /tmp/go.tgz; \
    rm /tmp/go.tgz; \
    export PATH="/golang/go/bin:${PATH}"; \
    go version

# The runtime container
FROM debian:buster-slim
WORKDIR /pulumi/projects

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
COPY --from=pulumi /pulumi/bin/pulumi /pulumi/bin/pulumi
COPY --from=pulumi /pulumi/bin/pulumi-language-go /pulumi/bin/pulumi-language-go
COPY --from=pulumi /pulumi/bin/pulumi-analyzer-policy /pulumi/bin/pulumi-language-go
ENV GOPATH=/go
ENV PATH "/pulumi/bin:${PATH}"

ENTRYPOINT ["pulumi"]
