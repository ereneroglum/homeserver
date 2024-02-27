# Homeserver automation

Homeserver automation to be used with an inter-network server behind NAT.

CAUTION: Deployment needs edits to paths in respective tasks file and edits to IP's in ```inventory``` and ```homer/config.yml```

## How to deploy

Use:

```bash
ansible-playbook --become --ask-become-pass --inventory inventory tasks.yaml
```

## About Homer and Caddy

Caddy is written with homer in mind and homer is in written with following services in mind:

- Jellyfin
- Komga
- Media
- Qbittorrent
