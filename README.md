# beingtomgreen.jellyfin_docker

A simple ansible role for running [Jellyfin](https://www.jellyfin.tv/) via Docker compose using the [LinuxServer.io Jellyfin image](https://docs.linuxserver.io/images/docker-jellyfin).

For additional information on how to best handle your media paths see information on [wiki.servarr.com](https://wiki.servarr.com/docker-guide#consistent-and-well-planned-paths) and [trash-guides.info](https://trash-guides.info/File-and-Folder-Structure/Hardlinks-and-Instant-Moves/).

## Installation

Given that Galaxy seems to have abandoned roles, I suggest referencing this repository directly in your projects `requirements.yml`:

```yaml
---

roles:
  - name: beingtomgreen.jellyfin_docker
    src: https://github.com/BeingTomGreen/ansible-role-jellyfin-docker.git

collections: []
```

You can then install it alongside your other requirements as normal:

```bash
ansible-galaxy install -r requirements.yml
```

## Example usage

### Basic usage

```yaml
---

- hosts: jellyfin
  become: true
  vars:
    jellyfin_docker_env_puid: 5000
    jellyfin_docker_env_pgid: 5000

    jellyfin_docker_media_volumes:
        - '/path/to/tv:/tv'
        - '/path/to/movies:/movies'
        - '/path/to/music:/music'
  roles:
    - role: beingtomgreen.jellyfin_docker
```
### Hardware Acceleration

Jellyfin supports hardware acceleration with a GPU, but will require [additional config](https://docs.linuxserver.io/images/docker-jellyfin/#hardware-acceleration), you should also look over the [Hardware Acceleration Enhancements](https://docs.linuxserver.io/images/docker-jellyfin/#hardware-acceleration-enhancements).

#### AMD

If you're rocking an AMD card, setup is nice and simple, as you'd expect:

```yaml
---

- hosts: jellyfin
  become: true
  vars:
    # Pass through your devices
    jellyfin_docker_devices:
      - '/dev/dri:/dev/dri'
      - '/dev/kfd:/dev/kfd'

    # Add the AMD mod
    jellyfin_docker_extra_environment_vars:
      DOCKER_MODS: 'linuxserver/mods:jellyfin-amd'
  roles:
    - role: beingtomgreen.jellyfin_docker
```

#### Intel

If you're rocking an Intel card, setup is also nice and simple:

```yaml
---

- hosts: jellyfin
  become: true
  vars:
    # Pass through your device
    jellyfin_docker_devices:
      - '/dev/dri:/dev/dri'

    # Add the AMD mod
    jellyfin_docker_extra_environment_vars:
      DOCKER_MODS: 'linuxserver/mods:jellyfin-opencl-intel'
  roles:
    - role: beingtomgreen.jellyfin_docker
```

#### NVIDIA

If you're stuck with an NVIDIA GPU, you will need the [container toolkit](https://github.com/NVIDIA/nvidia-container-toolkit) installed, and then set the `runtime` to `nvidia`:

```yaml
---

- hosts: jellyfin
  become: true
  vars:
    # Use the Nvidia container toolkit's runtime
    jellyfin_docker_runtime: 'nvidia'

    # Optionally, set a comma separated list of device UUIDs or index(es)
    jellyfin_docker_extra_environment_vars:
        NVIDIA_VISIBLE_DEVICES: '2,3'
  roles:
    - role: beingtomgreen.jellyfin_docker
```

#### ARM Devices

In most cases if `/dev/dri` exists on the host, it should just work:

```yaml
---

- hosts: jellyfin
  become: true
  vars:
    jellyfin_docker_devices:
      - '/dev/dri:/dev/dri'
  roles:
    - role: beingtomgreen.jellyfin_docker
```

If you're running a Raspberry Pi 4, be sure to enable `dtoverlay=vc4-fkms-v3d` in your `usercfg.txt`, and [check out the docs](https://docs.linuxserver.io/images/docker-jellyfin/#openmax-raspberry-pi).

### Want the kitchen sink?

Seriously, take a look at [`defaults/main.yml`](defaults/main.yml), it's obnoxiously commented, just for you.

```yaml
---

- hosts: jellyfin
  become: true
  vars:
    jellyfin_docker_cap_add:
      - CAP_WAKE_ALARM
      - CAP_AUDIT_CONTROL
    jellyfin_docker_cap_drop:
      - CAP_CHECKPOINT_RESTORE

    jellyfin_docker_container_name: 'jellyfin_container'

    jellyfin_docker_depends_on:
      - 'my-little service'

    jellyfin_docker_devices:
      - '/dev/dri:/dev/dri'
      - '/dev/kfd:/dev/kfd'

    jellyfin_docker_env_puid: 5000
    jellyfin_docker_env_pgid: 5000

    jellyfin_docker_env_timezone: 'Europe/London'

    jellyfin_docker_env_published_server_url: 'https://jellyfin.notplex.com'

    jellyfin_docker_extra_environment_vars:
      SOME_OTHER_VAR: 'A_VALUE'

    jellyfin_docker_image_tag: '10.10.6'

    jellyfin_docker_labels:
      traefik.enable: 'true'
      traefik.http.routers.traefik-https.entrypoints: 'https'

    jellyfin_docker_network_mode: 'host'

    jellyfin_docker_pull_policy: 'weekly'

    jellyfin_docker_restart_policy: 'always'

    jellyfin_docker_media_volumes:
      - '/path/to/tv:/tv'
      - '/path/to/movies:/movies'
      - '/path/to/music:/music'

    jellyfin_docker_base_directory: '/opt/jellyfin_docker'

    jellyfin_docker_prune_images: 'true'
    jellyfin_docker_prune_until: '24h'

  roles:
    - role: beingtomgreen.jellyfin_docker
```

## Role Variables

See [`defaults/main.yml`](defaults/main.yml) for more details.

## Requirements

- Docker & docker compose installed on the target host
- ansible_user will also need to be in the docker group

## Dependencies

- community.docker

## License

[MIT](LICENSE)

## Author Information

[Tom Green](https://github.com/BeingTomGreen)
