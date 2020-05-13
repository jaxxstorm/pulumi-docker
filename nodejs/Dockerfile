# syntax = docker/dockerfile:experimental
# Interim container so we can copy pulumi binaries
# Must be defined first
ARG PULUMI_VERSION=latest
ARG RUNTIME_VERSION=13.14.0
ARG PULUMI_IMAGE=jaxxstorm/pulumi
FROM ${PULUMI_IMAGE}:base.${PULUMI_VERSION} as pulumi

# The runtime container
FROM node:${RUNTIME_VERSION}-buster-slim
WORKDIR /pulumi/projects

# Install needed tools, like git
RUN --mount=target=/var/lib/apt/lists,type=cache \
    --mount=target=/var/cache/apt,type=cache \
    apt-get update -y && \
    apt-get install -y \
    git

# Uses the workdir, copies from pulumi interim container
COPY --from=pulumi /pulumi/bin/pulumi /pulumi/bin/pulumi
COPY --from=pulumi /pulumi/bin/*-nodejs* /pulumi/bin/
ENV PATH "/pulumi/bin:${PATH}"

ENTRYPOINT ["pulumi"]
