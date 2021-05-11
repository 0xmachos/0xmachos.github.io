---
title:		"Pi-hole, Unbound & Tailscale"
subtitle:	"How to configure a mesh VPN with your own DNS"
excerpt:	"Template excerpt yo."
tags:		[Guide]
---

I recently configured Pi-hole for DNS based ad blocking alongside Unbound for local recursive DNS resolution and then plugged it into Tailscale to enable access from anywhere. This is a quick guide for others and note to myself on how to achieve the same setup.

# Technologies Overview

* [Pi-Hole](https://pi-hole.net) "a DNS sinkhole that protects your devices from unwanted content, without installing any client-side software."
* [Unbound](https://www.nlnetlabs.nl/projects/unbound/about/) "validating, recursive, caching DNS resolver"
* [Tailscale](https://tailscale.com) "Zero config VPN. Installs on any device in minutes, manages firewall rules for you, and works from anywhere." built on [Wireguard](https://www.wireguard.com)

## Pi-hole

Pi-hole is a caching and forwarding DNS server. 

In the DNS hierarchy it's the lowest level of DNS server. When a device makes a DNS request Pi-hole will check if it has an response for that domain in its cache. If it does have a cached response it will reply to the device with it. However if it doesn't have a cached response it will forward the DNS request to whichever server(s) are configured as its upstream DNS servers. These upstream DNS servers are typically Google (`8.8.8.8`) or Cloudflare (`1.1.1.1`). In DNS vernacular these upstream servers are known as recursive resolvers, they will recursively traverse and query the DNS server hierarchy until get an authoritative response. 

For more, Cloudflare have a good explainer ["What is DNS?"](https://www.cloudflare.com/learning/dns/what-is-dns) on how these types of DNS servers fit together. 

### Why We Want Pi-hole

The thing that makes Pi-hole unique is it's web based User Interface (UI) and ability to load in block-lists. This is the sinkhole element of Pi-hole. Before it serves an IP address from its cache, or forwards it on, it consults block and allow lists. These lists enable us to block (sinkhole) certain domains so that the IP address returned to the requesting device is `0.0.0.0 ` (See: [Blocking mode](https://docs.pi-hole.net/ftldns/blockingmode/)). This prevents the requesting device from connecting to the blocked domain, thus blocking ads or whatever else was being served from that domain.

## Unbound

Unbound is a recursive DNS server or recursive resolver. As discussed above its job is to take a DNS request and get an authoritative response by recursively querying the hierarchy of DNS servers. You can read more about the types of DNS server in Cloudflare's ["What are the different types of DNS server?"](https://www.cloudflare.com/learning/dns/dns-server-types) article.

### Why We Want Unbound

If we configure Pi-hole to use Cloudflare (`1.1.1.1`) as our upstream DNS server then Cloudflare, or whichever upstream DNS we use, will have knowledge of every domain queried by our network. [Not great, not terrible]() depending on your threat model and privacy appetite.

<figure align="center">
	<img src="/img/posts/3-6-roentgen.gif" width="100%" alt="3.6 roentgen, not great, not terrible"/>
</figure>

A lot of data is served from Content Delivery Networks (CDNs). For domains which are served from CDNs, when you make a DNS request for the IP address of that domain, the response will be based on the physical location of the recursive resolver (upstream DNS server). This enables you to get the IP address of the CDN node which is physically closest to you. 

This is fine if the device initiating the DNS request is physically close to the recursive resolver. However if the recursive resolver is located in London and the device initiating the DNS request is located in Scotland the requesting device will likely be given CDN IP addresses also located in London. This is sub optimal as there may be CDN nodes located in Manchester or Motherwell which are physically closer to the requesting device. Ideally then the recursive resolver should be physically located on the network of the requesting devices.

The problem outlined above has already been considered by the folks who keep the Internet running. The solution they developed is called Client Subnet in DNS Queries (aka EDNS Client-Subnet or ECS) defined in [RFC 7871](https://datatracker.ietf.org/doc/rfc7871/). The recursive DNS resolver sends the requesting device's source aye IP subnet to the authoritative name server, this enables more accurate geolocation of the requesting device. The obvious issue here is that we're leaking more information to the authoritative name server. Which, again, depending on your threat model and privacy appetite, is not great, not terrible.

You can read more about ECS in Google's Public DNS article ["EDNS Client Subnet (ECS) Guidelines"](https://developers.google.com/speed/public-dns/docs/ecs), I also found this article ["EDNS Client-Subnet"](https://kazoo.ga/edns-client-subnet/) by Stephen Yip extremely useful.

It should be noted however that the use of Client Subnet in DNS Queries (ECS) and recursive resolver location proximity to requesting device may not have a large impact. This is thanks to AnyCast which resolves a domain to the same IP globally and requesting devices are directed to the nearest server at the network layer. Public DNS operators like Google and Cloudflare already deploy AnyCast. 

You can read more about AnyCast in Cloudflare's ["What is Anycast?"](https://www.cloudflare.com/en-gb/learning/cdn/glossary/anycast-network/) article.


## Tailscale

Tailscale is a control plane over the top of a Wireguard data plane. It's a mesh VPN, so rather than having a VPN gateway that every device connects to each device using Tailscale connects to one another using Wireguard tunnels. There's a lot of cool engineering under the hood to make Tailscale work that's beyond the scope of this blog post, I highly recommend reading the ["How Tailscale works"](https://tailscale.com/blog/how-tailscale-works/) blog post.

### Why We Want Tailscale

When I'm away from the network I've installed Pi-hole on I still want to be able to block ads so I need a way to connect back to the network to use the Pi-hole as my DNS server. 

I also want to be able to connect to other devices on that network. While Tailscales default is point to point connections you can turn devices which have the Tailscale client installed into [relay nodes](https://tailscale.com/kb/1019/subnets/) which allow you to to advertise whole subnets to all other devices on your Tailscale network.

# Configuration

## Hardware

* [Raspberry Pi 4](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/) 8GB of RAM
  * Running [Raspberry Pi OS Lite](https://www.raspberrypi.org/software/operating-systems/)
* [PRO Endurance microSD Memory Card 128GB](https://www.samsung.com/uk/memory-storage/memory-card/pro-endurance-microsd-card-with-sd-adapter-128gb-mb-mj128ga-eu/) (MB-MJ128GA/EU)

## Raspberry Pi Setup

Since this will be a headless host and only used for a few specific tasks I opted for Raspberry Pi OS Lite which comes in at less than 500MB. Along side this I downloaded the [official Raspberry Pi Imager](https://www.raspberrypi.org/software/) to install Raspberry Pi OS onto the Micro SD card.

## Enabling SSH on Raspberry Pi OS

By default SSH is disabled on Raspberry Pi OS. 

If you don't have a monitor and keyboard to connect to your Pi you'll need to use the computer you used to install Raspberry Pi OS onto the Micro SD card to create a file called `ssh` on `boot` volume of the Micro SD Card

On macOS you can execute:
```shell
touch /Volumes/boot/ssh
```

See the Raspberry Pi [SSH documentation](https://www.raspberrypi.org/documentation/remote-access/ssh/README.md) for other options.

## Configuring Raspberry Pi OS

Raspberry Pi OS is based on Debian, at the time of writing the Debian version is 10 (Buster).

Before you install Pi-hole, Unbound and Tailscale you should take some steps to harden the host. A few basics steps include:
1. Update the host
   * Then enable automatic updates via the [UnattendedUpgrades](https://wiki.debian.org/UnattendedUpgrades) package
2. Add a new `sudo` capable user and remove the default `pi` user
   * I have a script [`add_sudo_user`](https://github.com/0xmachos/bash-scripts/blob/master/add_sudo_user) which automates this
3. Configure SSH
   * See this [`sshd_config`](https://wiki.hacksoc.co.uk/networking/ssh#server) from the [AbertayHackers](https://twitter.com/abertayhackers) wiki
   * `sudo sshd -t`, `sudo systemctl restart sshd` and `sudo systemctl status sshd` are your friends for checking your `sshd_config` is sane and the `sshd` service is still running
4. Firewall
   * You probably only need ports `80/tcp` (Pi-hole Web UI), `53/udp` (DNS), `22/tcp` (SSH) and `41641/udp` (Tailscale)
   * You can use [Uncomplicated Firewall (`ufw`)](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29) to open only these ports

## Install Pi-Hole

See the Pi-hole [installation](https://docs.pi-hole.net/main/basic-install/) article or execute: 

```shell
curl -sSL https://install.pi-hole.net | bash
```

Before anyone bitches at me for recommending `curl | bash` at the end of the day are you going to download all the Pi-hole code and review it before you execute their install script with r00t privileges anyway and let their code handle a core part of your network? Probably not, 🤷‍♂️. 

## Install Unbound

Pi-hole have a [great guide](https://docs.pi-hole.net/guides/dns/unbound/) and ready to go [configuration file](https://docs.pi-hole.net/guides/dns/unbound/#configure-unbound). Highly recommend you go and follow their instructions instead of me reinventing the wheel here.

## Install Tailscale

Similarly Tailscale has an excellent guide ["Setting up Tailscale on Debian Buster"](https://tailscale.com/kb/1041/install-debian-buster/) which you should go and follow.

## Configuring Tailscale

If you install Tailscale on another device you should now be able to connect to your host running Pi-hole via its Tailscale IP address. You can test this by navigating to the Tailscale IP via a browser and adding `/admin` to access the Pi-hole web UI. 

### Pi-hole DNS Over Tailscale

To use the Pi-hole as the DNS server while using Tailscale we need to make a change to Tailscale and a change to Pi-hole. 

#### Tailscale

Navigate to the [DNS tab](https://login.tailscale.com/admin/dns) of the Admin console and add the Tailscale IP address of your Pi-hole host as a DNS server. You **cannot have [Magic DNS](https://tailscale.com/kb/1081/magic-dns/) enabled** for this work as far as I know. In my testing having Magic DNS enabled broke name resolution when I had a Tailscale IP or an IP on a relayed subnet as my DNS servers.

<figure align="center">
	<img src="/img/posts/pi-hole-tailscale-unbound/tailscale-dns.png" alt="My Tailscale DNS Configuration"/>
	<figcaption>My Tailscale DNS Configuration</figcaption>
</figure>


#### Pi-hole

By default Pi-hole listens for DNS queries on all interfaces but with a max distance of 1 hop, devices on the local network only. We need to adjust the "Pi-hole Interface listening behavior" to "Listen on all interfaces, permit all origins".

You can do this via the commandline by executing:

```shell
pihole -a interface all
```
Or via the web UI as seen below.

<figure align="center">
	<img src="/img/posts/pi-hole-tailscale-unbound/pi-hole-interface-settings.png" alt="Pi-hole Interface listening behavior web UI"/>
	<figcaption>Pi-hole Interface listening behavior web UI</figcaption>
</figure>

When I run `traceroute` from my Pi-hole host to the IP address of my iPhone connected to Tailscale it shows as 1 hop so I would expected the default option to work however in my testing DNS resolution only works when permit all origins is enabled. No idea why this happens.

As the Pi-hole web UI says do not enable this option if your Pi-hole host is exposed to the internet or you forward port `53` on your router to your Pi-hole host.

## Additional Tailscale Configuration

At present your connections will hit the Internet from a Tailscale IP address and you won't be able to access any of the other devices on the same network as your Pi-Hole host. 

You can enable the [Exit Node](https://tailscale.com/kb/1103/exit-nodes/) feature to hit the internet from the external IP address of the network your Pi-hole host is on and the [relay node](https://tailscale.com/kb/1019/subnets/) feature will allow you to advertise your local network to your Tailscale connected devices.

## Alternatives to Tailscale

If you want a similar setup with Pi-hole and Wireguard but don't want to get a third party involved you can very easily roll Wireguard using [trailofbits/algo](https://github.com/trailofbits/algo) via their [local installation instructions](https://github.com/trailofbits/algo/blob/master/docs/deploy-to-ubuntu.md).


# Conclusion

This is as much a note to myself as it is a post for others. Be careful when you edit SSH or firewall settings least you get locked out your box and have to start from scratch, I hear it fucking sucks especially on a Sunday.

I highly recommend the [Tailscale blog](https://tailscale.com/blog/) especially their latest post ["The Sisyphean Task Of DNS Client Config on Linux"](https://tailscale.com/blog/sisyphean-dns-client-linux/) which you're going to want to pour a stiff drink for because nobody should cry about DNS resolution while sober.

Since I've been banging on about DNS, I'd like to take this opportunity to pay tribute to [Dan Kaminsky](https://twitter.com/dakami) who sadly died on April 23rd 2021. I never knew him but I throughly enjoyed reading his work and the stories about him on Twitter after his death. You can find a wonderful obituary of him in the New York Times ["Daniel Kaminsky, Internet Security Savior, Dies at 42"](https://www.nytimes.com/2021/04/27/technology/daniel-kaminsky-dead.html) by Nicole Perlroth.

Thanks to [0xNige](https://twitter.com/0xNige) and [Sheldorr](https://twitter.com/Sheldorr) for reviewing this post.

<figure align="center">
	<img src="/img/dogs/dog4.jpg" alt="Pi-hole Interface listening behavior web UI"/>
	<figcaption>Photo by <a href="https://unsplash.com/@designbytholen">Regine Tholen</a> on <a href="https://unsplash.com">Unsplash</a></figcaption>
</figure>

