# magic-docker
Magic package manager for Mojo and MAX

This repository contains the docker configuration for the magic container image.
The magic container image is based on different base images, depending on the use case.
All images have magic installed in `/usr/local/bin/magic` and are ready to use.

---
## Pulling the images
The images are [available on "GHCR" (Github Container Registry)](https://github.com/modular/magic-docker/pkgs/container/magic).
You can pull them using docker like so:

```bash
docker pull ghcr.io/modular/magic:latest
```

There are different tags for different base images available:

- `latest` - based on `ubuntu:noble`
- `focal` - based on `ubuntu:focal`
- `bullseye` - based on `debian:bullseye`
- `noble-cuda-12.6.3` - based on `nvidia/cuda:12.6.3-ubuntu24.04`
- Some are initial Docker build images I am testing for a stealth project. For Modular's official Magic Docker images and repository, see the official [repository](https://github.com/modular/magic-docker)

---
## Usage with shell-hook

The following example uses the magic docker image as a base image for a multi-stage build.
It also makes use of the `shell-hook` feature of magic to define a convenient entry point (after executing the `shell-hook` script, the environment is activated.

```Dockerfile
FROM ghcr.io/modular/magic:latest AS build

# copy source code, pixi.toml and magic.lock to the container
COPY . /app
WORKDIR /app
# run some compilation / build task (if needed)
RUN magic run build
# run the `install` command (or any other). This will also install the dependencies into `/app/.magic`
# assumes that you have a `prod` environment defined in your pixi.toml
RUN magic run install -e prod
# Create the shell-hook bash script to activate the environment
RUN magic shell-hook -e prod > /shell-hook.sh

# extend the shell-hook script to run the command passed to the container
RUN echo 'exec "$@"' >> /shell-hook.sh

FROM ubuntu:24.04 AS production

# only copy the production environment into prod container
# please note that the "prefix" (path) needs to stay the same as in the build container
COPY --from=build /app/.magic/envs/prod /app/.magic/envs/prod
COPY --from=build /shell-hook.sh /shell-hook.sh
WORKDIR /app
EXPOSE 8000

# set the entrypoint to the shell-hook script (activate the environment and run the command)
# no more magic needed in the prod container
ENTRYPOINT ["/bin/bash", "/shell-hook.sh"]

CMD ["start-server"]
```
---
## Images

There are images based on `ubuntu`, `debian` and `nvidia/cuda` available.

### Ubuntu

The [`ubuntu:noble`](https://hub.docker.com/_/ubuntu) (24.04) based image is the default base image. It is used for the `latest` and `0.x.y` tag.

There are also images based on `ubuntu:focal` (20.04), `ubuntu:jammy` (22.04), `ubuntu:oracular` (24.10) and `ubuntu:plucky` (25.04) available.
These images use the tags `focal`, `0.x.y-focal`, ...

### Debian

Images based on [`debian:bullseye`](https://hub.docker.com/_/debian), `debian:bullseye-slim` (11), `debian:bookworm` and `debian:bookworm-slim` (12) are available.

These images have the tags `bullseye`, `0.x.y-bullseye`, ...

### NVIDIA/CUDA

Images based on [`nvidia/cuda`](https://hub.docker.com/r/nvidia/cuda) are available using the tags `cuda-<cuda-version>-jammy`, `cuda-<cuda-version>-focal`, `0.x.y-cuda-<cuda-version>-jammy`, ...

---

