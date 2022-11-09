+++ 
title = "Tailscale, DNS, and Self Hosted Services"
date = 2022-11-08
author = "Jeremy Borgman"
tags = ['Tailscale', 'DNS']
+++

## Introduction
This post walks through how to set up DNS on tailscale so it places nicely with
self hosted services. It assumes you are running AdGuard but a similar solution
should be available for any DNS service.

I recently deployed tailscale on my network and its been working well. I
installed it on my main server, my OPNSense Router, and all my edge devices. 

## OpnSense Setup
I start wireguard on the router with the following command:
```
tailscale up --reset --accept-dns=false --advertise-exit-node --advertise-routes=192.168.1.0/24
```

*--advertise-exit-node*  Allows client to funnel all traffic through this node

*--advertise-routes* Allows clients to reach other devices on the LAN not running tailscale.

This allows any device to funnel all traffic through my router. This is useful
when on public wifi. It also allows access to all the devices on my LAN even if
they do not have tailscale IP's.

Overall this has worked great, but I ran into a problem with DNS resolution on
any self hosted services when behind the VPN.

## DNS setup in Tailscale
Log in to tailscale and go to the DNS settings page. 
1. Enable MagicDNS
1. Override DNS
1. Set the global nameserver to the **Tailscale** IP address of your OpnSense
   Router

![Tailscale DNS Settings](/post/tailscale_and_dns/tailscale_dns.png)
## DNS Rewrites in AdGuard
Finally in order to resolve the DNS issues for clients funneling traffic through
tailscale, enable DNS rewrites in adguard.

![DNS Rewrite Menu](/post/tailscale_and_dns/rewrite.png)

![DNS Rewrite](/post/tailscale_and_dns/new_ip.png)


