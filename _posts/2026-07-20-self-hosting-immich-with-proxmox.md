---
title: Self Hosting Immich with Proxmox and ZFS
tags: selfhosting immich proxmox
style: fill
color: dark
description: Immich is a goolge photos alternative that is installed locally(self hosted) on your machine.
date: 2026-06-20
author: Yash Rathod
---

# Self Hosting Immich with Proxmox and ZFS

Google Photos was the last cloud service I hadn't replaced yet. Everything else in my life already runs on my own hardware — Frigate for cameras, AdGuard for DNS, a Homepage dashboard tying it all together. Photos felt like the harder problem because losing them is permanent. So I put it off longer than I should have.

Here's how I finally moved my photo library to Immich, running on a Proxmox VM with ZFS underneath, and why I picked that stack over the alternatives.

## Why Immich

I looked at PhotoPrism first. It's mature and lighter on resources, but the mobile app experience and face recognition aren't close to what Immich offers. Immich's backup app mirrors the Google Photos flow almost exactly — background upload, background sync, no manual intervention — which mattered more to me than saving a gigabyte of RAM.

The tradeoff: Immich is still moving fast. Breaking changes between versions aren't rare, and the project tells you outright to read the release notes before updating. I treat that as the cost of admission, not a dealbreaker.

## Why ZFS, not just ext4

My NAS storage already sits on ZFS, and I wasn't going to break that pattern for one VM. Three things sold me on keeping it:

- **Snapshots.** Before every Immich version bump, I snapshot the dataset. If a migration goes sideways, rollback is one command instead of restoring from backup.
- **Checksums.** Photos are the one dataset where silent bit rot actually costs you something you can't recreate. ZFS catches that; ext4 doesn't.
- **Send/receive.** Replicating the dataset to a second machine is a native feature, not a cron job wrapping rsync.

The cost is RAM — ZFS wants memory for the ARC cache — but on a homelab box that's already running Proxmox comfortably, it wasn't a real constraint.

## The setup

Proxmox host, one VM dedicated to Immich, storage backed by a ZFS pool passed through rather than virtualized on top of another filesystem. A few decisions that mattered:

**Separate the VM from the NAS.** I run Immich on its own VM rather than bolting it onto the same box doing Frigate's recording work. Photo processing (thumbnail generation, ML tagging, facial recognition) is bursty and CPU-heavy. I didn't want it competing with camera recording for cycles at 2am when a snapshot job kicks off.

**Dataset, not directory.** The Immich library lives on its own ZFS dataset, not just a folder inside a bigger one. That gives it its own snapshot schedule and its own quota, independent of everything else on the pool.

**Network segmentation already did the hard part.** Because my network is already split with OPNsense and VLANs behind the Cisco 2960-X, giving the Immich VM its own subnet with tightly scoped firewall rules was just extending a pattern I'd already built, not a new project.

## Backup strategy

Local snapshots protect against a bad update or a fat-fingered delete. They don't protect against the drive itself failing or the house burning down. My actual chain:

1. ZFS snapshot before every Immich update, kept for a rolling window
2. ZFS send/receive replication to a second pool on separate physical disks
3. An offsite copy, because two copies on one property is still one point of failure

If you're only doing step one, you don't have a backup — you have a slightly nicer undo button.

## Was it worth it

Six months in: yes, with one caveat. The upload experience from the phone app is genuinely as good as Google Photos was. Face grouping and search are close enough that I don't miss it. The caveat is maintenance — I read every changelog before updating, because Immich has broken migrations before and will again. That's a fair price for owning my own photo library outright, but it's not a "set it and forget it" service the way a NAS share is.

If you're already running Proxmox and have any ZFS storage in the house, adding Immich is a weekend project, not a rebuild. If you're starting from zero on both, budget more time for the storage layer than the app itself — Immich is the easy part.
