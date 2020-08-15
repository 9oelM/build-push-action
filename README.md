## About

GitHub Action to build and push Docker images.

___

* [Usage](#usage)
  * [Quick start](#quick-start)
  * [With Buildx](#with-buildx)
* [Customizing](#customizing)
  * [inputs](#inputs)
  * [outputs](#outputs)
* [Limitation](#limitation)

## Usage

### Quick start

```yaml
name: ci

on:
  pull_request:
    branches: master
  push:
    branches: master
    tags:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v2
      -
        name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          tags: |
            user/app:latest
            user/app:1.0.0
```

### With Buildx

You can also use our [setup-buildx](https://github.com/docker/setup-buildx-action) action that extends the
`docker build` command with the full support of the features provided by
[Moby BuildKit](https://github.com/moby/buildkit) builder toolkit to build multi-platform images.

```yaml
name: ci

on:
  pull_request:
    branches: master
  push:
    branches: master
    tags:

jobs:
  buildx:
    runs-on: ubuntu-latest
    steps:
      -
        name: Checkout
        uses: actions/checkout@v2
      -
        name: Set up QEMU
        uses: docker/setup-qemu-action@v1
        with:
          platforms: all
      -
        name: Set up Docker Buildx
        id: buildx
        uses: docker/setup-buildx-action@v1
        with:
          install: true
      -
        name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          builder: ${{ steps.buildx.outputs.builder }}
          platforms: linux/amd64,linux/arm/v6,linux/arm/v7,linux/arm64,linux/386,linux/ppc64le,linux/s390x
          tags: |
            user/app:latest
            user/app:1.0.0
```

## Customizing

### inputs

Following inputs can be used as `step.with` keys

| Name                | Type    | Default                           | Description                        |
|---------------------|---------|-----------------------------------|------------------------------------|
| `context`           | String  | `.`                               | Build's context is the set of files located in the specified `PATH` or `URL` |
| `file`              | String  | `./Dockerfile`                    | Path to the Dockerfile. |
| `build-args`        | String  |                                   | Newline-delimited list of build-time variables |
| `labels`            | String  |                                   | Newline-delimited list of metadata for an image |
| `tags`              | String  |                                   | Newline-delimited list of tags **required** |
| `pull`              | Bool    | `false`                           | Always attempt to pull a newer version of the image |
| `target`            | String  |                                   | Sets the target stage to build |
| `no-cache`          | Bool    | `false`                           | Do not use cache when building the image |
| `builder`**¹**      | String  |                                   | Builder instance |
| `platforms`**¹**    | String  |                                   | Comma-delimited list of target platforms for build |
| `load`**¹**         | Bool    | `false`                           | Shorthand for `--output=type=docker` |
| `push`              | Bool    | `false`                           | Whether to push the built image (or shorthand for `--output=type=registry` if buildx used) |
| `outputs`**¹**      | String  |                                   | Newline-delimited list of output destinations (format: `type=local,dest=path`) |
| `cache-from`**¹**   | String  |                                   | Newline-delimited list of external cache sources (eg. `user/app:cache`, `type=local,src=path/to/dir`) |
| `cache-to`**¹**     | String  |                                   | Newline-delimited list of cache export destinations (eg. `user/app:cache`, `type=local,dest=path/to/dir`) |

> **¹** Only available if [docker buildx](https://github.com/docker/buildx) is enabled.
> See [setup-buildx](https://github.com/docker/setup-buildx-action) action for more info.

### outputs

Following outputs are available

| Name          | Type    | Description                           |
|---------------|---------|---------------------------------------|
| `digest`      | String  | Image content-addressable identifier also called a digest |

## Limitation

This action is only available for Linux [virtual environments](https://help.github.com/en/articles/virtual-environments-for-github-actions#supported-virtual-environments-and-hardware-resources).
