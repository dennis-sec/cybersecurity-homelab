# Phase 2 — Network Bridges

## Goal

Create the virtual switching that the lab networks sit on, and decide where routing responsibility lives.

## What was built

Created `vmbr1` as the lab switch. The deliberate decision here is that Proxmox does **not** route for the lab — pfSense does. Putting an IP and NAT on the bridge would have created a second router competing with pfSense for the same job, which is a classic source of confusing, hard-to-diagnose network behaviour. So the bridge is left as a plain Layer 2 switch and pfSense provides all routing, NAT, DHCP and DNS.

## A correction worth recording (the config evolved)

The bridge was originally created with no IP at all, which matched the "Proxmox is not a router" decision cleanly.

Later, so the Proxmox host itself could reach the Wazuh manager on the lab network, the host was given a single address on `vmbr1` (`10.10.10.2`) — with the gateway field deliberately left **blank**. This is an important distinction: giving the host an address on the segment makes it *reachable* on that network, but leaving the gateway blank means it does not become a router and does not gain a second default route. The host's own route to the internet still goes via `vmbr0`, and pfSense still provides all routing, NAT, DHCP and firewalling for the lab.

The one consequence is that machines on the lab segment can now reach the Proxmox host at `10.10.10.2`. Every machine on that segment is trusted (Kali, Pi-hole, Wazuh), and the deliberately vulnerable target lives on a *separate* segment (Phase 8) with firewall rules that give it no path to the host at all. Locking down host access properly is a hardening item.

## State at end of phase

A lab switch in place, with pfSense — not Proxmox — owning all routing for the lab.
