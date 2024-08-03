# Homeserver automation

Homeserver automation to be used with an inter-network server behind NAT.

## What to change

First change server IP in ```inventory``` file.

Then change IP addresses in ```homer/config.yml```. Now you can deploy the configuration excpet ```recyclarr.yaml```.

After deploying media stack, change IP and API Keys in ```recyclarr/recyclarr.yml```. After deployment, manually set up Samba and Syncthing.

## How to deploy

Generally to use a playbook run:

```bash
ansible-playbook --become --ask-become-pass --inventory inventory playbook.yaml
```

General order of deployment is:

- Edit ```inventory```

Now deploy these for reverse proxy and DNS:

- Homer (You need to edit ```homer/config.yml``` before this step)
- Caddy
- Pi-hole

Then edit your `/etc/hosts` and add `<IP of Server> pihole.internal` to your configuration computers (the one you execute ansible, not the remote server) `/etc/hosts`.

Then every entry in `caddy/Caddyfile` as an A record in the pihole (including `pihole.internal`). Now you can remove the `<IP of Server> pihole.internal` from the configuration computers `/etc/hosts`.

Setup your dhcp server such that the advertised dns points to the pihole.

Setup `systemd-resolved` in `uplink` mode in your server pointing to itselfs lan ip (probably something like `192.168.0.x`, you can obtain it with `ip a`).

After deploy:

In any order:

- Jellyfin
- Komga
- Qbittorrent
- Quassel
- Home Assistant

In order (1st Group):

- Media
- Recyclarr (You need to edit ```recyclarr/recyclarr.yml``` before this step)

In order (2nd Group):

- Rsshub
- Freshrss

On every device on the network, go to `certificates.internal` and download and install `root.crt` as a Certificate Authority.

## About Dependencies

There are several dependency chains in this configuration. If a file is not listed in dependencies, it can be deployed by itself.

### Caddy - Reverse proxy

Caddy depends on:

- Homer

### Homer - Dashboard

Homer depends on:

- Jellyfin
- Komga
- Media
- Qbittorrent
- Nextcloud

If you do not want any of the services listed above, find and remove their configuration from ```homer/config.yml```.

### Recyclarr

Homer depends on:

- Media

## Creating a user on Quassel

Firstly ssh into your server. Following commands should be run on the server.

Firstly stop Quassel with. ``` sudo systemctl stop container-quassel```. Then use following to create a temprory container for adding a user

```
sudo podman run --rm --tty –-interactive \
  --name=quassel-core-temprory \
  -e PUID=1001 \ # Edit this to match your media user in other .yml files
  -e PGID=1001 \ # Edit this to match your media user in other .yml files
  -e TZ=Europe/Istanbul \ # Edit this to match your media user in other .yml files
  -e DB_BACKEND=SQLite \
  -e AUTH_AUTHENTICATOR=Database \
  -v /mnt/storage/Quassel:/config \ # Edit this to match your media user in other .yml files
  --entrypoint ash \
  linuxserver/quassel-core:latest
```

Then add a user with:

```
quasselcore --configdir=/config --add-user
```

Exit the shell with ```exit``` and start your container with ```sudo systemctl start container-quassel```.

## Finding admin password of Qbittorrent

You can find password of both Qbittorrent instances by:

```
sudo podman logs qbittorrent
sudo podman logs qbittorrent-manual
```

## Reverse proxy on home assistant

Add following to `homeassistant/configuration.yaml`:


```
http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 10.0.0.0/8
```
