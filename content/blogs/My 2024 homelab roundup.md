---
date: 2025-01-05
# image: ""
lastmod: 2025-01-05
showTableOfContents: false
title: "My 2024 Homelab Roundup"
type: "post"
tags: [homelab, homeserver, self-host, network]
---

Hello World! Welcome to my first blog. I hope you had a great 2024, and I'm looking forward to an even better 2025!

This post will go over my homelab setup (the what's and the why's) at the end of 2024. It will cover all of the hardware, software, networking, services etc.

## Network

Let's start with my network.

### Hardware

The following hardware runs the network in my home:
1. Zyxel MG-105 - 2.5G switch
2. TP Link SG-1005P - 1G POE+ switch
3. Zyxel NWA50AX WiFi AP
4. ISP ONT

Notice how there's no hardware for the router and firewall? That's because Opnsense runs in a virtual machine, which is my router/firewall of choice.

The basic layout of my network:

![Network Layout](</images/Network_layout.jpg>)

### Software

1. Opnsense: My router/ firewall of choice because it is Free and Open Source (FOSS), has a ton of plugins and gets updated frequently while still being rock stable. Have had 0 downtime since I installed it (except for when I broke it ofc :p)

2. Using the stock firmware on the NWA50AX. Might flash Openwrt once it's warranty ends

3. Pi-hole: DNS server that blocks ads, tracking and malicious domains

### VLANs

I have 3 VLANs configured as of now:

1. VLAN(A) for trusted devices/ servers. All the phones, pcs, laptops, trusted services in the home are part of this VLAN
2. VLAN(B) for internet facing services
3. Isolated VLAN(C) for untrusted services and testing purposes

VLAN A can access servers in VLANs B & C, while VLANs B & C cannot access any devices in other VLANs.

### WiFi

Have 2 SSIDs. Both on 2.4GHz and 5GHz.

1. Primary: All (trusted) devices that need access to my home server connect to this
2. Secondary: A guest network that all untrusted/ guest devices connect to. Also devices that do not need access to the homeserver connect to this. Has L2 isolation so no devices can talk to each other

## Home Server

### Hardware

A single computer takes care of all my services.

Specifications as follows:
* CPU: i3 12100 - got it because I felt it was more than enough for all my needs. Got it over the 12100F because I wanted an iGPU as the build does not have a dGPU.
* RAM: 64GB (32*2) 4800 MT/s DDR5 - initially started off with 8GB single channel, then later upgraded to 16GB dual channel. After hosting a ton of services, I was running short on memory so upgraded to 32GB single channel. Even that was somehow less (thanks to local AI models and partially ZFS caching(learnt that later, it's okay if your RAM is almost full, that's how ZFS works)), upgraded to 64GB dual channel. 4800MT/s because that's the max the CPU's IMC (integrated memory controller) can run memory at
* PSU: MSI MAG A650GF
* SSD: Samsung EVO 970 Plus 1TB
* HDDs: 3 * 2TB Seagate Ironwolf Pro - needed enterprise grade NAS drives because the homeserver runs 24*7. Didn't want to cheap out on these because they store all the important data

### Software

The homeserver runs Proxmox (a baremetal hypervisor). All the services run either as containers or virtual machines.

All the 3 HDDs are part of a ZFS pool formatted as RAIDZ1. This means I can lose up to 1 drive without losing any data. The array was created on proxmox itself.

### Services

#### VMs

1. Opnsense: I decided to virtualise it because of a few reasons. Them being:
	* It is easier to restore the VM snapshot when something goes wrong. Trust me something will go wrong when you are tinkering with a lab setup. This has already saved my time multiple times in the past. If I had decided to run it on bare metal, I would have had to waste time either to reinstall it or reset it and then restore my config or something else along those lines
	* My NIC gets virtualised so I don't have to deal with drivers in FreeBSD (Opnsense)
	* There is hardly performance penalty these days in virtualising, so why not. There are no downsides to virtualising it, at least I don't see any. If you know of any, please let me know
	
	The whole of internet technically runs on virtualised hardware so why not this.

2. OpenMediaVault: OpenMediaVault is my NAS software of choice. It's a very simple NAS software unlike TrueNAS or UNRAID but I don't need anything complex either. It's just supposed to enable SMB on the RAID array. This VM also runs a couple of docker containers:
	* Nextcloud: An alternative to Google drive, Dropbox etc.
	* Collabora office: A Microsoft office/ Google office suite replacement. Integrated into Nextcloud so files are stored there.
	* Jellyfin: A media server used to view self owned media
	* Postgres: SQL for Nextcloud
	* Redis: In memory cache for Nextcloud
	* Portainer: Set it up just to play around with it

#### Containers

##### Local services

1. Pi Hole: DNS server that blocks ads, tracking and malicious domains

2. Caddy: A reverse proxy that fetches SSL/ TLS certificates. All services hosted have a public certificate because I don't want to deal with the headache of private certificates

	Pro Tip: Use/ request wildcard certificate for your home/ private network so that even if someone goes on CTL (certificate transparency log), they cannot map your internal network.

3. Homarr: My dashboard. Still WIP. Never found time to complete it

4. AI:
	* Ollama: To run AI models. So far have only run LLMs because I do not have a beefy GPU
	* Open WebUI: Front end for Ollama. Looks and feels like any other LLM interface. Recently set it up to search with my searx instance. Still playing around RAGs and exploring

5. Firefly III: I am interested in personal finance too and always wanted to track and see where I spend my money. Did not want to put in too much effort of remembering and manually categorising the spends. So set up Firefly III just a couple of months back and it does most of the things I want. Have rules to auto categorise the spends and it also has an option to add recurring transactions. Maybe overkill for what I want, but almost perfect for all my needs

6. SearXNG: A self hosted meta search engine. Recently set it up to use it along with my local AI instance

7. Vaultwarden: A self hosted password manager. It is an unofficial server implementation of Bitwarden password manager

##### Internet facing services

1. Nextcloud: This instance of nextcloud is exposed to the internet and used for the following amongst others:
	* To share files between friends and family (instead of using Google drive, Dropbox etc.)
	* Nextcloud cospend as an alternative to splitwise

2. Authentik: It is a self hosted SSO used for managing users

3. Navidrome: A self hosted music player

4. Nginx Proxy Manager: A reverse proxy for the internet facing services. No particular reason for using it over caddy other than to learn and explore


## Looking Forward to 2025

This is just a snapshot of my homelab setup at the end of 2024. I've learned a lot and had a blast experimenting with different technologies. In 2025, I plan to continue exploring new ideas, expand my knowledge, and share more of my experiences and learnings through this blog!
