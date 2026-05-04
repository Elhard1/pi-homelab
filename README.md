# 🏠 Pi Homelab

A fully automated Raspberry Pi 5 home server setup. One script gets you a complete self-hosted homelab — start with a beautiful dashboard that links to everything, then add smart home, automation, local AI, ad blocking, and more.

> **First time?** The setup script installs everything and opens your Homepage dashboard automatically. From there you can access every service without memorising any ports.

---

## ✨ What You Get

| Service | Purpose | Port |
|---|---|---|
| [Homepage](https://github.com/gethomepage/homepage) | **Your launchpad — links to everything** | 3005 |
| [Portainer](https://github.com/portainer/portainer) | Docker management UI | 9443 |
| [Home Assistant](https://github.com/home-assistant/core) | Smart home hub | 8123 |
| [n8n](https://github.com/n8n-io/n8n) | Workflow automation | 5678 |
| [Pi-hole](https://github.com/pi-hole/pi-hole) | Network-wide ad blocking | 8053 |
| [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager) | Reverse proxy | 81 |
| [Vaultwarden](https://github.com/dani-garcia/vaultwarden) | Self-hosted password manager | 8080 |
| [Uptime Kuma](https://github.com/louislam/uptime-kuma) | Service monitoring | 3001 |
| [Nextcloud](https://github.com/nextcloud/server) | Self-hosted cloud storage | 8181 |
| [Stirling PDF](https://github.com/Stirling-Tools/Stirling-PDF) | PDF tools | 8888 |
| [Ollama](https://github.com/ollama/ollama) | Local LLM engine | 11434 |
| [Open WebUI](https://github.com/open-webui/open-webui) | Local AI chat interface | 3000 |
| [Watchtower](https://github.com/containrrr/watchtower) | Auto-update containers | - |
| [Tailscale](https://tailscale.com) | Secure remote access from anywhere | - |
| [Netdata](https://github.com/netdata/netdata) | Real-time system monitoring | 19999 |

---

## 🛒 Requirements

### Hardware
- Raspberry Pi 5 (4GB or 8GB recommended — 8GB ideal for running local AI)
- MicroSD card (32GB+ recommended — use a quality card like SanDisk Extreme; cheap cards cause filesystem corruption under Docker write loads)
- USB-C power supply (5V 5A / 25W — the official Pi 5 supply is strongly recommended; underpowered supplies cause random reboots)
- Ethernet or WiFi connection

> ⚠️ **Storage tip:** SD cards wear out faster than SSDs under constant Docker writes. For a long-running server, consider booting from a USB SSD or NVMe HAT instead of SD card. See the [SD to SSD migration guide](docs/migrate-to-ssd.md) to move your setup across with zero reinstallation.

### Software (on your PC/Mac)
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- SSH client — PuTTY (Windows) or Terminal (Mac/Linux)

---

## 🚀 Quick Start

### Step 1 — Flash the OS

1. Open [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
2. Select **Raspberry Pi 5** as the device
3. Click **OS** → scroll down to **Raspberry Pi OS (other)** → select **Raspberry Pi OS Lite (64-bit)** for a headless server, or **Raspberry Pi OS (64-bit)** if you plan to attach a screen later
4. Select your SD card as storage
5. Click **Next** → **Edit Settings** and configure:
   - Hostname: `homelab` (or anything you prefer, no spaces)
   - Username and password (remember these!)
   - WiFi name and password
   - Enable SSH ✓
6. Flash the card, insert into your Pi, and power on
7. Wait **2–3 minutes** for first boot (the Pi expands the filesystem and restarts once — this is normal)

> 💡 **Tip:** The Raspberry Pi Imager handles all formatting automatically — do not format the SD card manually beforehand.

### Step 2 — Connect via SSH

**Windows (PowerShell):**
```powershell
ssh yourusername@homelab.local
```

**Mac/Linux (Terminal):**
```bash
ssh yourusername@homelab.local
```

The first time you connect it will ask you to confirm the host fingerprint — type `yes` and press Enter.

> 💡 **If `homelab.local` doesn't resolve:** Check your router's connected devices list for your Pi's IP address and SSH using the IP directly: `ssh yourusername@192.168.x.x`

### Step 3 — Run the setup script

```bash
curl -fsSL https://raw.githubusercontent.com/Elhard1/pi-homelab/main/setup.sh | bash
```

The script will prompt you for:
- Your hostname
- Pi-hole admin password
- n8n admin password
- Tailscale authentication (a URL will appear — open it in your browser)

Then it handles everything else automatically. Go make a coffee — it takes 10–20 minutes depending on your internet speed.

---

## 🏠 Step 4 — Open Homepage (Your Dashboard)

Once the script finishes, open your browser and go to:

```
http://homelab.local:3005
```

or using your Pi's IP:

```
http://YOUR_PI_IP:3005
```

This is your **central dashboard** — it has clickable tiles for every service so you never need to remember ports again. Bookmark this page.

> 💡 **Homepage troubleshooting:** If you see a "Host validation failed" error, the `HOMEPAGE_ALLOWED_HOSTS` environment variable needs to include your exact hostname and IP. The setup script handles this automatically, but if you set it up manually make sure it contains both `homelab.local:3005` and `YOUR_PI_IP:3005` with **no spaces** around the comma.
>
> If you want to access Homepage over Tailscale, also add your Tailscale IP: `100.x.x.x:3005`

---

## 📋 What the Script Does

1. Updates all system packages
2. Installs Docker and Docker Compose
3. Disables Docker IPv6 (fixes image pull failures on some networks)
4. Adds your user to the Docker group
5. Installs Tailscale for secure remote access
6. Installs Netdata for real-time system monitoring
7. Creates the homelab directory and Docker Compose stack
8. Configures Homepage dashboard with all service links pre-loaded
9. Pulls all Docker images
10. Starts all services
11. Prints a full summary of service URLs

---

## 🔒 Remote Access with Tailscale

Tailscale creates a secure private network between your devices — no port forwarding, no exposed ports, no public IP needed.

### Why Tailscale?
- Works through **Starlink and CGNAT** (which don't give you a real public IP, making traditional VPNs like WireGuard difficult to set up)
- Zero config — install and authenticate, that's it
- End-to-end encrypted
- Free tier supports up to 100 devices
- Works on iOS, Android, Windows, Mac, Linux

### Setup

**On the Pi (done automatically by the script):**
```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Open the URL it gives you, sign in with Google/GitHub/Microsoft, and your Pi will appear in your Tailscale network.

**On your other devices:**
1. Download Tailscale from [tailscale.com/download](https://tailscale.com/download)
2. Sign in with the same account
3. Your Pi will appear in the device list

**Find your Pi's Tailscale IP:**
```bash
tailscale ip -4
```

This gives you a `100.x.x.x` address. Use this to access your homelab from anywhere:

```
http://100.x.x.x:3005   — Homepage dashboard from anywhere in the world
http://100.x.x.x:8123   — Home Assistant remotely
http://100.x.x.x:5678   — n8n remotely
```

**Add your Tailscale IP to Homepage allowed hosts:**

After getting your Tailscale IP, update your compose file:
```bash
nano ~/homelab/docker-compose.yml
```

Find `HOMEPAGE_ALLOWED_HOSTS` and add your Tailscale IP with no spaces around the comma:
```yaml
- HOMEPAGE_ALLOWED_HOSTS=homelab.local:3005,192.168.x.x:3005,100.x.x.x:3005
```

Then redeploy:
```bash
cd ~/homelab && docker compose up -d homepage
```

### Check Tailscale status
```bash
tailscale status
```

---

## 🛡️ Pi-hole Network Ad Blocking

Pi-hole blocks ads, trackers, and malware domains for **every device on your network** — no browser extensions needed, works on phones, smart TVs, and apps.

### Enable network-wide blocking

1. Find your Pi's IP: `hostname -I`
2. Log into your router admin page (usually `192.168.0.1` or `192.168.1.1`)
3. Find DNS settings (under LAN, DHCP, or Advanced settings depending on your router)
4. Set **Primary DNS** to your Pi's IP address
5. Save and restart your router

All devices will now use Pi-hole automatically without any changes on each device.

Access the dashboard at `http://homelab.local:8053/admin`

> 💡 **Note:** Pi-hole cannot block YouTube ads because Google serves ads from the same domains as video content. For YouTube ad blocking you still need a browser extension like uBlock Origin.

---

## 🤖 Local AI with Ollama

Run AI models completely locally — no cloud, no data leaving your network.

**Pull your first model:**
```bash
docker exec -it ollama ollama pull llama3.2
```

Then open **Open WebUI** from your Homepage dashboard or at `http://homelab.local:3000`

**Recommended models for Pi 5 8GB:**
| Model | Size | Speed | Notes |
|---|---|---|---|
| `llama3.2` | ~2GB | Fast | Best starting point |
| `phi3` | ~2.3GB | Fast | Great for coding |
| `mistral` | ~4GB | Medium | Good general purpose |
| `llama3.1:8b` | ~4.7GB | Slow | More capable but slower on Pi |

> ⚠️ 7B+ models run at 1–2 tokens/second on Pi 5 — usable but slow. 3B models run at 4–8 tokens/second which feels reasonable. The 8GB RAM is the key advantage here.

---

## 💾 Migrating from SD Card to SSD

Running on SD card but want better reliability and performance? You can clone your entire running setup — all containers, volumes, config, and data — to an SSD with no reinstallation required.

See the full guide: **[docs/migrate-to-ssd.md](docs/migrate-to-ssd.md)**

---

## 🔧 Service Setup Checklist

After your dashboard is up, set up each service in this order:

- [ ] **Homepage** `http://homelab.local:3005` — Bookmark this first!
- [ ] **Portainer** `https://homelab.local:9443` — Create admin account (accept the self-signed cert warning)
- [ ] **Nginx Proxy Manager** `http://homelab.local:81` — Create your admin account on first login
- [ ] **Home Assistant** `http://homelab.local:8123` — Create account, add devices
- [ ] **n8n** `http://homelab.local:5678` — Change default password immediately
- [ ] **Uptime Kuma** `http://homelab.local:3001` — Add monitors for each service
- [ ] **Vaultwarden** `http://homelab.local:8080` — Requires HTTPS (set up Nginx Proxy Manager first)
- [ ] **Nextcloud** `http://homelab.local:8181` — Create admin account
- [ ] **Open WebUI** `http://homelab.local:3000` — Create account, pull a model
- [ ] **Pi-hole** `http://homelab.local:8053/admin` — Point router DNS to Pi IP
- [ ] **Netdata** `http://homelab.local:19999` — No setup needed, just open it

---

## ⚠️ Known Issues & Lessons Learned

These are real issues encountered during setup and how to fix them.

### Docker image pull fails with IPv6 error
```
dial tcp [2606:4700::...]:443: connect: network is unreachable
```
**Fix:** Disable Docker IPv6:
```bash
sudo nano /etc/docker/daemon.json
```
Add:
```json
{
  "ipv6": false,
  "ip": "0.0.0.0"
}
```
```bash
sudo systemctl restart docker
cd ~/homelab && docker compose pull && docker compose up -d
```

### Homepage shows "Host validation failed"
Homepage requires the `HOMEPAGE_ALLOWED_HOSTS` environment variable to include the exact host:port you're accessing it from.
```yaml
environment:
  - HOMEPAGE_ALLOWED_HOSTS=homelab.local:3005,192.168.x.x:3005
```
> ⚠️ No spaces around the comma — `host1:port,host2:port` not `host1:port, host2:port`

After editing the compose file:
```bash
docker compose down homepage && docker compose up -d homepage
```

### Portainer crashes with `SIGILL: illegal instruction`
Some Portainer image versions have ARM64 compatibility issues on Pi 5. Use the alpine tagged version:
```yaml
image: portainer/portainer-ce:2.21.5-alpine
```

### SSH connection refused on first boot
- Wait longer — first boot takes 2–3 minutes (Pi restarts once while expanding filesystem)
- Check your router for the Pi's IP and SSH directly to the IP instead of the hostname
- Verify WiFi credentials were saved correctly in the imager

### Pi reboots repeatedly
- **Power supply** — most common cause. Pi 5 needs a genuine 25W supply. Phone chargers cause random reboots.
- **SD card quality** — cheap cards fail under Docker write loads. Use SanDisk Extreme or better.
- **SD card filesystem corruption** — if you see `EXT4-fs error: bad block bitmap checksum` in `sudo dmesg`, your SD card has bad blocks. Reflash or switch to USB SSD — see [migrate-to-ssd.md](docs/migrate-to-ssd.md).

### Vaultwarden shows "You are not using a secure context"
Vaultwarden requires HTTPS to function. Set up Nginx Proxy Manager with a proxy host pointing to Vaultwarden and enable SSL before using it.

### `.local` hostname not resolving on Windows
Windows mDNS can be unreliable. Use the Pi's direct IP address instead:
```powershell
ssh yourusername@192.168.x.x
```
Find the IP from your router's connected devices list.

---

## 📁 File Structure

```
pi-homelab/
├── setup.sh                      # Main automated setup script
├── docker-compose.yml            # Full Docker stack
├── config/
│   └── homepage/
│       └── services.yaml         # Homepage dashboard config
├── docs/
│   ├── manual-setup.md           # Step-by-step manual setup guide
│   └── migrate-to-ssd.md         # Clone SD card setup to SSD
└── README.md
```

---

## 📦 Updating Services

Watchtower automatically updates all containers at 4am daily.

Manually update all services:
```bash
cd ~/homelab && docker compose pull && docker compose up -d
```

Update a single service:
```bash
cd ~/homelab && docker compose pull <service-name> && docker compose up -d <service-name>
```

---

## 🛠️ Useful Commands

```bash
# View all running containers and their status
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# View logs for a service
docker logs <container-name> --tail 50

# Restart a single service
docker restart <container-name>

# Stop all services
cd ~/homelab && docker compose down

# Start all services
cd ~/homelab && docker compose up -d

# Check disk usage
df -h

# Check system resources
htop

# Check Tailscale status and connected devices
tailscale status

# Get Tailscale IP
tailscale ip -4

# Pull an Ollama model
docker exec -it ollama ollama pull llama3.2

# List installed Ollama models
docker exec -it ollama ollama list
```

---

## 🙏 Attributions

This project uses the following open source projects:

- [Homepage](https://github.com/gethomepage/homepage) — MIT License
- [Portainer CE](https://github.com/portainer/portainer) — zlib License
- [Home Assistant](https://github.com/home-assistant/core) — Apache 2.0 License
- [n8n](https://github.com/n8n-io/n8n) — Sustainable Use License
- [Pi-hole](https://github.com/pi-hole/pi-hole) — EUPL License
- [Nginx Proxy Manager](https://github.com/NginxProxyManager/nginx-proxy-manager) — MIT License
- [Vaultwarden](https://github.com/dani-garcia/vaultwarden) — AGPL-3.0 License
- [Uptime Kuma](https://github.com/louislam/uptime-kuma) — MIT License
- [Nextcloud](https://github.com/nextcloud/server) — AGPL-3.0 License
- [Stirling PDF](https://github.com/Stirling-Tools/Stirling-PDF) — MIT License
- [Ollama](https://github.com/ollama/ollama) — MIT License
- [Open WebUI](https://github.com/open-webui/open-webui) — MIT License
- [Watchtower](https://github.com/containrrr/watchtower) — Apache 2.0 License
- [Tailscale](https://tailscale.com) — BSD 3-Clause License
- [Netdata](https://github.com/netdata/netdata) — GPL-3.0 License
- [rpi-clone](https://github.com/billw2/rpi-clone) — GPL-2.0 License

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.
