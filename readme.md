<h1 align="center">umbrelOS 2.0<br />
<div align="center">
<a href="https://github.com/DelChris/umbrel"><img src=".github/header.png" title="Logo" style="max-width:100%;" width="256" /></a>
</div>
<div align="center">

[![Build]][build_url]
[![Release]][release_url]

</div></h1>

Docker container of [umbrelOS](https://umbrel.com/umbrelos) 2.0, an OS for self-hosting.

> [!WARNING]
> **Beta.** This is a fork of [dockur/umbrel](https://github.com/dockur/umbrel) (umbrelOS 1.x) ported to umbrelOS 2.0. It works well in our tests, but it has not been validated on many hosts yet. Back up your data before upgrading.

## Features ✨

- Runs umbrelOS 2.0 inside a Docker container, without dedicated hardware or a virtual machine
- Installs and runs Umbrel apps on the host Docker daemon
- Runs virtual machines (Umbrel Machines) with libvirt and QEMU, accelerated by KVM when available
- Available for `amd64` and `arm64` (`arm64` is built but not tested yet)

## Usage 🐳

##### Docker Compose:

```yaml
services:
  umbrel:
    image: ghcr.io/delchris/umbrel:beta
    container_name: umbrel
    pid: host
    privileged: true
    ports:
      - 80:80
      - 443:443
      - 2000:2000
    volumes:
      - ./umbrel:/data
      - /var/run/docker.sock:/var/run/docker.sock
    restart: always
    stop_grace_period: 1m
```

See [compose.yml](compose.yml) for a commented version.

##### Docker CLI:

```bash
docker run -d --name umbrel --pid=host --privileged -p 80:80 -p 443:443 -p 2000:2000 -v "${PWD:-.}/umbrel:/data" -v "/var/run/docker.sock:/var/run/docker.sock" --restart always --stop-timeout 60 ghcr.io/delchris/umbrel:beta
```

## FAQ 💬

### How do I upgrade from dockurr/umbrel 1.x?

  1. Stop the container: `docker compose down`
  2. Back up the whole data folder (for example `./umbrel`). umbrelOS 2.0 updates app configurations on its first start, so going back to 1.x requires this backup.
  3. Replace the image with `ghcr.io/delchris/umbrel:beta` and add the `privileged`, `443` and `2000` lines shown above.
  4. Start it again: `docker compose up -d`

  Keep the same `/data` folder and the default `umbrel_main_network` subnet (`10.21.0.0/16`). Your apps are started again automatically.

### Do I need `privileged: true`?

  Only for Machines. libvirt needs it to create the virtual network and QEMU uses `/dev/kvm`. Without it, umbrelOS runs normally and hides Machines.

### How do I change the storage location?

  Change the bind mount of `/data`, for example:

  ```yaml
  volumes:
    - /srv/umbrel:/data
  ```

  Use an absolute path or a path relative to the compose file. App data lives in the same folder.

  If a folder inside it is a symbolic link to another disk (for example `home` pointing to a storage pool), also bind mount the target of the link at the same path:

  ```yaml
  volumes:
    - /srv/umbrel:/data
    - /mnt/storage:/mnt/storage
  ```

### Which images does umbrelOS remove?

  umbrelOS periodically removes unused app images. In this container it only removes images it downloaded itself, never other images on the host.

### Does it work on Docker Desktop (Windows, macOS)?

  Yes, including Machines on Windows with WSL2. The Docker Desktop kernel lacks a few networking features, so machines run without anti-spoofing filters and 3D graphics are rendered in software.

### How do I build the image myself?

  ```bash
  docker build -t umbrel .
  ```

  The build downloads umbrelOS from [getumbrel/umbrel](https://github.com/getumbrel/umbrel) and applies the patches in [patches](patches).

## Credits 🙏

Based on [dockur/umbrel](https://github.com/dockur/umbrel) by [Kroese](https://github.com/kroese). umbrelOS is developed by [Umbrel](https://umbrel.com) and distributed under its own [license](https://github.com/getumbrel/umbrel/blob/master/LICENSE.md).

[build_url]: https://github.com/DelChris/umbrel/actions/workflows/build.yml
[release_url]: https://github.com/DelChris/umbrel/releases

[Build]: https://github.com/DelChris/umbrel/actions/workflows/build.yml/badge.svg
[Release]: https://img.shields.io/github/v/release/DelChris/umbrel?include_prereleases&color=066da5
