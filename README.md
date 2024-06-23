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

In any order:


- Jellyfin
- Komga
- Nextcloud
- Qbittorrent
- Quassel

In order (1st Group):

- Media
- Homer (You need to edit ```homer/config.yml``` before this step)
- Caddy
- Recyclarr (You need to edit ```recyclarr/recyclarr.yml``` before this step)

In order (2nd Group):

- Rsshub
- Freshrss

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
