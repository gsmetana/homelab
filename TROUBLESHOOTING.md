# Troubleshooting

## General
So you were messing around and broke everything. Nice. Here are some things to try before everyone gets mad that services are down

1. Check VMs

First, make sure that the Proxmox VMs are running. Even if the reverse proxy on the manager VM is busted, you can still log into the dashboard at `:8006`

2. Check Docker/Komodo

Komodo periphery is running in systemd, so it's unlikely to have any issues (`systemctl --type=service --state=running` should show periphery.service). The komodo core dashboard should be accessible on the manager VM at `:9120`

If it's not, `cd /etc/homelab-komodo` (https://github.com/gsmetana/homelab-komodo) and try to bring it up. Misconfigured secrets (`my.core.config.yaml`) shouldn't prevent access at `:9120`, but network/swarm issues might. The command used to create the overlay network on the manager was `docker network create --driver overlay --attachable myNetwork`

## Networking

One way to break everything is to change the server IP addresses. Here is a (hopefully) comprehensive list of everywhere you need to update with new configuration.

1. `/etc/homelab-komodo/my.core.config.yaml`

Multiple services access the postfix relay running on the manager VM with the variable `EMAIL_HOST`

2. Komodo Core (`Resources -> Servers -> Config`)

Each server is running periphery at `:8120`.

3. Docker swarm

If the manager VM changes IP, you need to destroy and recreate the overlay network. If the network is working, non-manager nodes running traefik-kop should have no issues pushing updates to `traefik-redis:6379`

