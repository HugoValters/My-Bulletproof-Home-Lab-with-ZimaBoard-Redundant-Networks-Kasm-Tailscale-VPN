# Building a Bulletproof Home Lab: Redundant Networks, Kasm, and Tailscale on Zima Board

Imagine this: Your primary internet connection drops dead in the middle of a critical task. No dedicated IP? No problem. What if you could seamlessly switch to a backup network, access your secure home lab from anywhere — even your iPad on the road — and do it all without exposing your entire setup to risks? That’s the magic I’m unpacking today with a Zima Board, dual networks, Kasm workspaces, and Tailscale VPN. Stick around; by the end, you’ll have the blueprint to make your home lab unbreakable. And trust me, the payoff is worth every step.

## Why This Setup Changes Everything

Not everyone has the luxury of a static IP or enterprise-grade redundancy. But in a world where downtime means lost productivity (or worse, missed opportunities), why settle? I set out to create a fully functional, redundant home lab using a compact Zima Board. In my case, it’s powered by an optical fiber network (192.168.1.0/24) as primary and a 5G backup (192.168.118.0/24). If the optical line fails, 5G kicks in automatically — smooth as silk.

This isn’t just theory. I’ve tested it during weekend blackouts, and the handover is flawless. Plus, with Tailscale, you get secure remote access without port-forwarding headaches. Open it to the public DNS for quick checks, or lock it down to your devices only. It’s flexible, secure, and insanely convenient for travelers like me who need to vet suspicious links or keep Discord running from an iPad.

**Warning:** All IPs, Tailscale accounts, and Kasm installs in this guide were created for demonstration. They’ve been wiped clean post-video. Always prioritize security — don’t expose more than necessary.

If you’re running a single network, skip ahead to the Kasm install. But if redundancy calls your name, let’s dive in.

YouTube Video: [YouTube Video on Building a Bulletproof Home Lab: Redundant Networks, Kasm, and Tailscale on Zima Board](https://www.youtube.com/watch?v=Wj6I6KFwl6s&t).

## Step 1: Prep Your Zima Board with Ubuntu

Out of the box, Zima Board ships with CasaOS. For this powerhouse setup, we need Ubuntu. If you haven’t switched yet, check out my install guide [Install Ubuntu on Zima Board/Blade Video Tutorial](https://www.youtube.com/watch?v=sKwpDblye0). Once Ubuntu is up, we’re ready to rock.

## Step 2: Configuring Redundant Networks for Zero Downtime

Here’s where the intrigue builds: Two independent networks, one smart failover.

First, inspect your interfaces:
```
ip a
```

You’ll spot something like:
* enp3s0: Optical network (e.g., IP 192.168.1.135)
* enp2s0: 5G network (e.g., IP 192.168.118.9)

Install the tools for priority assignment:
```
sudo apt install -y ifmetric ifplugd
```

Assign metrics (lower = higher priority):
```
sudo ifmetric enp2s0 200
sudo ifmetric enp3s0 100
```

Verify with:
```
ip r
```

You should see routes prioritizing the optical network:
```
default via 192.168.1.254 dev enp3s0 proto dhcp src 192.168.1.135 metric 100
default via 192.168.118.1 dev enp2s0 proto dhcp src 192.168.118.9 metric 200
...
```

For faster switching, tweak ifplugd. Edit /etc/default/ifplugd with nano:
Change:
```
ARGS="-q -f -u0 -d10 -w -I"
```

To:
```
ARGS="-q -f -u0 -d2 -w -I"
```

Save (Ctrl+X, Y, Enter), then enable:
```
systemctl enable ifplugd && systemctl start ifplugd
```

Boom — network swaps now happen in seconds. Test by yanking a cable; watch the magic.

## Step 3: Installing Kasm for Your Virtual Workspaces
With networks solid, let’s add Kasm — the ultimate browser-based desktop environment. It’s like having isolated VMs on demand, perfect for testing or secure browsing.

Run this one-liner:
```
cd /tmp && curl -O https://kasm-static-content.s3.amazonaws.com/kasm_release_1.17.0.bbc15c.tar.gz && tar -xf kasm_release_1.17.0.bbc15c.tar.gz && sudo bash kasm_release/install.sh
```

Grab a coffee; it takes ~10 minutes. Kasm handles everything. Save your credentials securely — you’ll need them for login.

## Step 4: Layering On Tailscale for Secure, Anywhere Access
Now the fun part: Tailscale turns your lab into a private mesh VPN or also possible to create a public site. Sign up at tailscale.com (straightforward process), then install:
```
curl -fsSL https://tailscale.com/install.sh | sh
```

Authenticate:
```
tailscale up
```

Follow the browser link to confirm your device. You’ve got your first node!
Enable HTTPS in your Tailscale dashboard for certificates.
Before going public, shift Kasm’s ports to avoid conflicts. Navigate to:
```
cd /opt/kasm/1.17.0/docker
```

Edit start (or similar config file) to change port 443 to 8443. Restart Kasm:
```
cd /opt/kasm/bin && ./stop && ./start
```

Disable UFW for simplicity (or configure specific ports if you’re paranoid):
```
systemctl stop ufw && ufw disable
```

Generate certs (replace with your Tailscale subdomain, e.g., kasm.tailffcb92.ts.net):
```
tailscale cert kasm.tailffcb92.ts.net
```

Expose ports:
```
sudo tailscale serve --bg https+insecure://localhost:443
sudo tailscale funnel --bg 8443
sudo tailscale funnel --bg 443
sudo tailscale funnel --bg https+insecure://localhost:8443
```

Confirm in your browser. Vola — your Kasm is accessible via Tailscale’s DNS from anywhere, no client needed. Need to lock it down? Install Tailscale on your devices for device-only access.

To revoke public access:
```
sudo tailscale funnel --https=443 off && sudo tailscale funnel --https=8443 off
```

## The Grand Reveal: A Fully Redundant, Secure Home Lab
There you have it — a Zima Board fortress with failover networks, isolated Kasm sessions, and Tailscale’s effortless VPN. I’ve used this to check malicious links on the go or keep workflows uninterrupted during outages. It’s not just functional; it’s liberating.

Pro tip: Install Tailscale on your laptop, phone, or tablet for max security. Abuse it? Nah — use it wisely.

What’s next? More Tailscale scenarios in upcoming posts. If this sparked your inner tinkerer, drop a clap, share your tweaks in the comments, or follow for more home lab hacks. Your unbreakable setup awaits — what are you waiting for?

This post is based on my video tutorial. Not sponsored by Tailscale or Kasm, but hey, collaborations welcome! All rights reserved by Hugo Valters
[YouTube Video on Building a Bulletproof Home Lab: Redundant Networks, Kasm, and Tailscale on Zima Board](https://www.youtube.com/watch?v=Wj6I6KFwl6s&t).

Follow for more:<br>
X.com: https://x.com/hugovalters<br>
bsky.app: https://bsky.app/profile/hugovalters.bsky.social<br>
YouTube: https://www.youtube.com/@hugovalters<br>
Homepage: https://www.valters.eu<br>
GitHub: https://github.com/hugovalters<br>
GitLab: https://gitlab.com/hugovalters<br>
Medium: https://blog.valters.eu
