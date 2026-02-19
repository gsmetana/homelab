# homelab


## Overview

The primary server on the LAN is a VM called `nas`. It has a RAID controller (passthrough) for storage of photos, videos, music, and documents. Docker services serve this data.

The second server on the LAN is a VM called `mgr`. It runs services that generally have lower hardware requirements but are more critical (annoying if they go down). This includes the reverse proxy (Traefik), authentication (Authentik), and password management (Vaultwarden).

A backup machine called `minipc` is located offsite. This hosts a Proxmox Backup Server to snapshot VMs.


## Connectivity

Tailscale is installed directly on Proxmox. This Proxmox to securely transmit data the backup server by using the static IP provided by Tailscale or "MagicDNS" using the host names.

Docker instances in the LAN are part of a swarm, although "swarm mode" is not used to orchestrate the containers; all services run standalone. The swarm is used purely to create an overlay network and simplify DNS configuration of the reverse proxy. Currently, traefik-kop is used to help route traffic from the single Traefik instance on `mgr` to services running on `nas`.

## Backup

The Proxmox UI is used to configure VM backups. Files on RAID are selectively backed up using a Proxmox backup client running as a Docker service.

## Docker (Komodo)

