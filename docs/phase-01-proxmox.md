# Phase 1 — Proxmox Host

## Goal

Stand up the hypervisor that everything else runs on — a single Lenovo ThinkCentre M920s (i5-8500, 24 GB RAM) — and get a working desktop on it for local access.

## What was built

Installed Proxmox VE 9.1 on the M920s. Added a lightweight XFCE desktop so the host can be driven locally as well as over the web UI.

## Problem solved — the deb822 repo format

Proxmox 9.1 uses the newer deb822 source format for APT repositories. Disabling a repository in this format is not done by commenting out the components line, as with the old format — that throws "malformed entry" errors on every `apt` run. The correct way is to set `Enabled: false` on the repository. Worth recording because the failure mode (repeated malformed-entry errors) doesn't obviously point at the fix.

## Trade-off recorded

The host runs with root auto-login for convenience. This widens the local attack surface — anyone with physical access lands on a root session. It's an accepted trade-off for a single-operator lab, and proper auth hardening (a non-root admin, disabling auto-login) is on the roadmap. Documented rather than hidden, because a reviewer should see the decision was deliberate.

## State at end of phase

A working Proxmox host with a local desktop, ready to run the lab VMs.
