# Sov-Stack

**Sov-Stack**, short for "sovereign stack" is a solution to the sovereign computing problem for the somewhat technically inclined, especially with a good working knowledge of Docker, or willingness to learn along the way. Host your entire life yourself, starting with Bitcoin and Lightning node, Tor proxy, SyncThing file server, password manager, Nostr relay and more, on almost any hardware, be it x86 or ARM. It sits somewhere in the middle between "turn-key" solutions such as Umbrel, myNode, RaspiBlitz or Start9 and "DIY" approaches such as RaspiBolt. In fact, it draws most of its inspiration from RaspiBolt but instead of you going through all the installation procedures yourself, you get `Dockerfile`s for most services that you can inspect yourself. Your computer then builds these images locally (which includes building most of the services from source as well) and you bring them up as needed.

# Services

These services are currently available:
* [Tor](https://www.torproject.org/) proxy (used by other services but also exposed on port 9050 for your own use)
* [Tailscale](https://tailscale.com/) client (for accessing your services on the go, also available as exit node for your personal VPN)
* [Bitcoin Core](https://github.com/bitcoin/bitcoin) daemon (bitcoind)
* [electrs](https://github.com/romanz/electrs), a blockchain indexing service

Services planned to be included:
* Lightning Network node + management frontend(s)
* Blockchain explorer
* SyncThing server
* Nostr relay
* Password manager

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
