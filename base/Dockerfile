# syntax = docker/dockerfile:experimental
FROM ubuntu:bionic AS builder
ARG PULUMI_VERSION=latest
RUN apt-get update -y && \
      apt-get install -y \
      curl \
      git \
      build-essential \
      git
# Install the Pulumi SDK, including the CLI and language runtimes.
RUN if [ "$PULUMI_VERSION" = "latest" ]; then \
    curl -fsSL https://get.pulumi.com/ | bash; \
  else \
    curl -fsSL https://get.pulumi.com/ | bash -s -- --version $PULUMI_VERSION ; \
  fi

# The runtime container
# This is our base container, so let's copy all the runtimes to .pulumi/bin
FROM ubuntu:bionic
WORKDIR /pulumi
COPY --from=builder /root/.pulumi/bin bin

ENV PATH "/pulumi/bin:${PATH}"
ENTRYPOINT ["pulumi"]
