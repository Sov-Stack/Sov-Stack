# Sov-Stack

**Sov-Stack**, short for "sovereign stack" is a sovereign computing solution for somewhat technically inclined users, especially with a good working knowledge of Docker, or willingness to learn along the way.

Host your entire digital life starting with Bitcoin and Lightning node, Tor proxy, SyncThing file server, password manager, Nostr relay and more, on almost any hardware, be it x86 or ARM.

In terms of technical complexity, it sits somewhere in the middle between "turn-key" solutions such as Umbrel, myNode, RaspiBlitz or Start9 and "DIY" approaches such as RaspiBolt. In fact, it draws most of its inspiration from RaspiBolt but instead of you going through all the installation procedures yourself, you get `Dockerfile`s for most services that you can inspect yourself. Your computer then builds these images locally (which includes building most of the services from source as well) and you bring them up as needed.

# Services

The following is a list of services available as part of the stacks, split into categories. Containers are classified by their source:

| Container | Description |
|-----------|-------------|
| ![Source](https://img.shields.io/badge/Source-darkgreen) | There is a bespoke `Dockerfile` available in this repository and the service is built from source code. |
| ![Binary](https://img.shields.io/badge/Binary-556611) | There is a bespoke `Dockerfile` available in this repository but the image uses pre-built binaries. (Must still be FOSS.) |
| ![Pre-built](https://img.shields.io/badge/Pre--built-554411) | Uses a pre-built image from registry, usually provided by a vendor. `Dockerfile` source must be available and service itself be FOSS. |

| Service | Description | Container | Notes |
|---------|-------------|-----------|-------|
| **Networking** |
| [`tor`](https://torproject.org/) | Onion-routing proxy for privacy | ![Source](https://img.shields.io/badge/0.4.7.13-Source-darkgreen?logo=git&logoColor=white) |
| [`tailscale`](https://tailscale.com/) | Mesh VPN client w/ exit node | ![Pre-built](https://img.shields.io/badge/latest-Pre--built-554411?logo=docker&logoColor=white) | Client itself is FOSS (WireGuard) but Tailscale backend is proprietary (?). |
| [`caddy`](https://github.com/caddyserver/caddy) | A simple web server. | ![Pre-built](https://img.shields.io/badge/latest-Pre--built-554411?logo=docker&logoColor=white) | Used as reverse-proxy with SSL termination for other services. |
| **Monitoring**|
| [`cadvisor`](https://github.com/google/cadvisor) | Monitoring daemon for the host and containers | ![Pre-built](https://img.shields.io/badge/0.47.1-Pre--built-554411?logo=docker&logoColor=white) |
| [`bitcoin-prometheus-exporter`](https://github.com/jvstein/bitcoin-prometheus-exporter) | Prometheus exporter for Bitcoin nodes | ![Pre-built](https://img.shields.io/badge/latest-Pre--built-554411?logo=docker&logoColor=white) |
| **Bitcoin** |
| [`bitcoind`](https://github.com/bitcoin/bitcoin) | Bitcoin Core node | ![Source](https://img.shields.io/badge/25.1-Source-darkgreen?logo=git&logoColor=white) |
| [`electrs`](https://github.com/romanz/electrs) | Blockchain indexing service | ![Source](https://img.shields.io/badge/0.9.13-Source-darkgreen?logo=git&logoColor=white) |
| [`jam`](https://github.com/joinmarket-webui/jam-docker) | A 2-in-1 image with [JoinMarket](https://joinmarket.net/) (CoinJoin service) and JAM (frontent) | ![Pre-built](https://img.shields.io/badge/latest-Pre--built-554411?logo=docker&logoColor=white) |
| **Lightning Network** |
| [`lnd`](https://github.com/lightningnetwork/lnd) | Lightning Network Daemon | ![Source](https://img.shields.io/badge/v0.17.3--beta-Source-darkgreen?logo=git&logoColor=white) |
| [`thunderhub`](https://github.com/apotdevin/thunderhub) | Browser-based LND manager | ![Pre-built](https://img.shields.io/badge/v0.13.31-Pre--built-554411?logo=docker&logoColor=white) |
| [`rtl`](https://github.com/Ride-The-Lightning/RTL/) | Browser-based LND/CLN/Eclair manager | ![Pre-built](https://img.shields.io/badge/0.15.1-Pre--built-554411?logo=docker&logoColor=white) |
| **Miscellaneous** |
| [`syncthing`](https://syncthing.net/) | Continuous file synchronization | ![Pre-built](https://img.shields.io/badge/latest-Pre--built-554411?logo=docker&logoColor=white) |
| [`vaultwarden`](https://github.com/dani-garcia/vaultwarden) | A lightweight Bitwarden-compatible password manager server | ![Pre-built](https://img.shields.io/badge/latest-Pre--built-554411?logo=docker&logoColor=white) |

Services planned to be included:
* Lightning Network node + management frontend(s)
* Blockchain explorer
* Nostr relay
* Prometheus + Grafana

# Getting started

TODO: This section will eventually be refactored into a complete guide, but those interested in the project and getting started, these are roughly the steps needed. Previous experience in setting up something like RaspiBolt and using Docker are required to follow these instructions for now.

The steps to get started are as follows:
* Get a machine to run this on. Something like a Raspberry Pi 4B will work, I also tested it on an Orange Pi 5. Any x86 system (old laptops or business minicomputers) should do as well. You will need an SSD or NVMe drive and I recommend getting 2TB as Docker does consume quite a lot of space to build its images (you can get by with 1TB but you will need to do periodical cleanups).
* Operating system does not matter, you should be fine with Raspberry Pi OS, I recommend DietPi if you don't plan to run anything else on this machine (everything in Sov-Stack is containerized). Even Windows with Docker in WSL or Docker Desktop should theoretically work.
* Install Docker with the Compose plugin.
* Checkout this repository and `cd` into it.
* Inspect the `docker-compose.yml` file - for now it assumes all the services will have a subdirectory under `/data`, so make those directories and change their ownership to user `1000` (which is most likely the main non-root user on your OS).
* Copy the `.env.sample` file to `.env` and review, fill in settings (for now these are only for Tailscale, so skip this if you don't use it).
* Install configuration file for Tor: `cp tor/torrc.sample /data/tor/cfg/torrc`
* Start Tor with `docker compose up tor.sov -d --build`
* Install configuration file for Bitcoin Core: `cp bitcoind/bitcoin.conf.sample /data/bitcoind/data/bitcoin.conf`
* Edit `/data/bitcoind/data/bitcoin.conf` and enable last two lines for initial block download (give it more memory in `dbcache` if you wish). Optionally enable clearnet.
* Start Bitcoin Core with: `docker compose up bitcoind.sov -d --build`
* After it builds and starts, wait possibly serveral days for it to sync. You can at any point shell into it with `docker exec -it bitcoind.sov sh` and use `bitcoin-cli` to monitor progress.
* Stop Bitcoin Core with: `docker stop bitcoind.sov` (this might take a good few minutes as it flushes the cache to disk)
* Edit `/data/bitcoind/data/bitcoin.conf` and disable the last two lines.
* Start Bitcoin Core again with `docker compose up bitcoind.sov -d`
* Start electrs with `docker compose up electrs.sov -d --build` (this takes several hours to sync as well)
* If you've set up Tailscale key and name of your node in `.env`, you can also bring it up: `docker compose up tailscale.sov -d` - `bitcoind`'s P2P port 8333 and `electrs`' 50001 will be available via the node's Tailscale IP and you can also use it as exit node to route traffic e.g. through your home when you're on the go.
