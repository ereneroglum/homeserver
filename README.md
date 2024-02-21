# Homeserver automation

Homeserver automation to be used with an inter-network server behind NAT.

CAUTION: Deployment needs edits to paths in ```tasks.yaml``` and edits to IP's in ```inventory``` and ```homer/config.yml```

## How to deploy

Use:

```bash
ansible-playbook --become --ask-become-pass --inventory inventory tasks.yaml
```
