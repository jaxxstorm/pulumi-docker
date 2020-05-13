# Pulumi Docker images

This is an *highly experimental* repo that builds pulumi images for each language only SDK. The aim is to reduce the docker image size considerably.

The pulumi official docker image is quite large because it has to bundle all the SDKs we support:

  - Go
  - Python
  - NodeJS
  - DotNet

However, it's almost certainly the case that you don't _need_ all these languages when you're running your pulumi program. These images are language specific. They contain the `pulumi` binary, the `pulumi` language runtime for that SDK and any additional language quirks.

The following is a quick table comparing the image sizes when compressed:

| Image | Size |
| --- | --- |
| **pulumi/pulumi** | ~725MB |
| jaxxstorm/pulumi:base.latest | ~83MB |
| jaxxstorm/pulumi:go.latest | ~206MB |
| jaxxstorm/pulumi:nodejs.latest | ~120Mb |
| jaxxstorm/pulumi:dotnet.latest | ~279Mb |
| jaxxstorm/pulumu:python.latest | ~114MB |


**Please note, while I'm currently an employee of Pulumi, this isn't currently officially support by Pulumi - use it at your own risk**

## Images

We build a matrix of images for differing Pulumi versions and language SDKs.

### Base Image

The base image just contains the pulumi binaries and language runtimes, but _not_ the SDK runtimes. If you use the base image, you'll have to install Go/Python/Dotnet/NodeJS yourself. The image format is:

```
jaxxstorm/pulumi:base.<pulumi_version>
```

### SDK Images

Images with the SDK runtimes are generated in the following format:

```
jaxxstorm/pulumi:<sdk>.<pulumi_version>-<sdk_runtime_version>
```

## Usage

None of these images have `CMD` or entrypoint set, so you'll need to specify the commands you want to run, for example:

```
docker run -e PULUMI_ACCESS_TOKEN=<TOKEN> -v "$(pwd)":/pulumi/projects $IMG /bin/bash -c "npm ci && pulumi preview -s <stackname>"
```

This decision has been made to foster flexibility in both CI and local systems. 

*This will probably changed based on feedback from the community*

The `WORKDIR` is set to `/pulumi/projects`, so if you're using volumes, please mount your project there.

### Acknowledgements

Thanks to [@jen20](http://github.com/jen20) for the awesome docker build action



## Considerations

These images _do not_ include additional tools you might want to use when running a pulumi provider. For example, if you're using the [pulumi-kubernetes](https://github.com/pulumi/pulumi-kubernetes) with [Helm](https://helm.sh/), you'll need to use these images as a base image, or install the `helm` command as part of your CI setup.

## Building

To try and keep the image size down, we're using:

  - [Multi-stage builds](https://docs.docker.com/develop/develop-images/multistage-build/)
  - [Buildkit](https://docs.docker.com/develop/develop-images/build_enhancements/)

With that in mind, if you want to build one of these images yourself, you'll need to enable buildkit, like so:

```
DOCKER_BUILDKIT=1 docker build --build-arg PULUMI_VERSION=2.1.1 -t jaxxstorm/pulumi:base.latest ./base
```
