# Building a Bulletproof Home Lab: Redundant Networks, Kasm, and Tailscale on Zima Board

Imagine this: Your primary internet connection drops dead in the middle of a critical task. No dedicated IP? No problem. What if you could seamlessly switch to a backup network, access your secure home lab from anywhere—even your iPad on the road—and do it all without exposing your entire setup to risks? That's the magic I'm unpacking today with a Zima Board, dual networks, Kasm workspaces, and Tailscale VPN. Stick around; by the end, you'll have the blueprint to make your home lab unbreakable. And trust me, the payoff is worth every step.

## Why This Setup Changes Everything

Not everyone has the luxury of a static IP or enterprise-grade redundancy. But in a world where downtime means lost productivity (or worse, missed opportunities), why settle? I set out to create a fully functional, redundant home lab using a compact Zima Board. In my case, it's powered by an optical fiber network (192.168.1.0/24) as primary and a 5G backup (192.168.118.0/24). If the optical line fails, 5G kicks in automatically—smooth as silk.

This isn't just theory. I've tested it during weekend blackouts, and the handover is flawless. Plus, with Tailscale, you get secure remote access without port-forwarding headaches. Open it to the public DNS for quick checks, or lock it down to your devices only. It's flexible, secure, and insanely convenient for travelers like me who need to vet suspicious links or keep Discord running from an iPad.

**Warning:** All IPs, Tailscale accounts, and Kasm installs in this guide were created for demonstration. They've been wiped clean post-video. Always prioritize security—don't expose more than necessary.

If you're running a single network, skip ahead to the Kasm install. But if redundancy calls your name, let's dive in.

## Step 1: Prep Your Zima Board with Ubuntu

Out of the box, Zima Board ships with CasaOS. For this powerhouse setup, we need Ubuntu. If you haven't switched yet, check out my install guide (linked in the video above). Once Ubuntu is up, we're ready to rock.

## Step 2: Configuring Redundant Networks for Zero Downtime

Here's where the intrigue builds: Two independent networks, one smart failover.

First, inspect your interfaces:
```
ip a
```
