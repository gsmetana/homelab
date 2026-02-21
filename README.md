# homelab


## Overview

The primary server on the LAN is a VM called `nas`. It has a RAID controller (passthrough) for storage of photos, videos, music, and documents. Docker services serve this data.

The second server on the LAN is a VM called `mgr`. It runs services that generally have lower hardware requirements but are more critical (annoying if they go down). This includes the reverse proxy (Traefik), authentication (Authentik), and password management (Vaultwarden).

A backup machine called `minipc` is located offsite. This hosts a Proxmox Backup Server to snapshot VMs.


### Connectivity

Tailscale is installed directly on Proxmox. This Proxmox to securely transmit data the backup server by using the static IP provided by Tailscale or "MagicDNS" using the host names.

Docker instances in the LAN are part of a swarm, although "swarm mode" is not used to orchestrate the containers; all services run standalone. The swarm is used purely to create an overlay network and simplify DNS configuration of the reverse proxy. Currently, traefik-kop is used to help route traffic from the single Traefik instance on `mgr` to services running on `nas`.

### Backup

The Proxmox UI is used to configure VM backups. Files on RAID are selectively backed up using a Proxmox backup client running as a Docker service.

### Docker (Komodo)

The Komodo manager node runs the Core service and database. Other nodes only need to install the periphery service. All Komodo services should have unique container names. If the Komodo containers are in the same Docker overlay network, they should be able to communicate using container names without exposing any ports

Komodo is used to manage secrets, but no sensitive configuration is located in this repo. The `secrets` section of `core.config.toml` (located elsewhere) describes environment variables for containers. These are referenced in Komodo with brackets for interpolation, eg `[[AUTHENTIK_SECRET_KEY]]` and populated into .env files during deployment.

## Instructions

To setup a new server machine:

1. Install Proxmox
2. Install Tailscale

Add repo:
```
curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.noarmor.gpg | sudo tee /usr/share/keyrings/tailscale-archive-keyring.gpg >/dev/null
curl -fsSL https://pkgs.tailscale.com/stable/debian/trixie.tailscale-keyring.list | sudo tee /etc/apt/sources.list.d/tailscale.list
```
Install:
```
sudo apt-get update
sudo apt-get install tailscale

sudo tailscale up
```

3. Connect to Proxmox Backup Server
4. Create Debian VM
5. Install Docker on Debian VM

Add repo:
```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg lsb-release

curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
Install:
```
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```
6. Configure Docker to run on startup
```
sudo systemctl enable docker
```
7. Join Docker Swarm

On a manager node, run this command to get a join token
```
docker swarm join-token worker
```
9. Install Komodo

10. (If manager) Komodo Resource Sync 

## TODO
- Document SSH key distribution & root login


   
   
