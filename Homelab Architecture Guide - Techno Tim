# 🚀 Ultimate Homelab Architecture Guide (2026 Edition)

This guide breaks down a highly robust homelab setup featuring over 50 self-hosted services, covering network topology, hypervisors, storage, and application stacks ranging from media to DevOps.

## 🕸️ Network & Topography
A flat, secure network is the backbone of the lab, utilizing VLANs to keep traffic isolated and organized.
* **Gateway to Switches:** ISP feeds into a primary gateway, passing down to core switches and then directly to servers.
* **VLAN Segregation:**
    * **Management:** Trusted network for hypervisors (Proxmox), UniFi network devices, and infrastructure that needs cross-VLAN access.
    * **Main (Trusted):** Daily driver devices like workstations, laptops, mobile phones, and Apple TVs/HomePods.
    * **IoT:** Strictly locked down network for smart lights, thermostats, and devices lacking an OS that can be securely updated. These devices cannot communicate outside of this VLAN without specific ACLs.
    * **Cameras:** Dedicated purely to security cameras and the NVR.
* **Colocation Expansion:** Three remote servers host public-facing services (like high-traffic documentation sites) and communicate with the local home network via a secure VPN for private management and off-site backups.

## 🖥️ Hypervisors & High Availability (HA)
Instead of relying on the hypervisor to manage high availability, HA is pushed down to the application layer.
* **Proxmox VE:** 6 total nodes (3 at home, 3 in the colocation). Used purely as the virtualization layer.
* **Application-Layer HA:** Instead of clustering VMs for HA at the hypervisor level, redundant LXC containers run across different nodes. If a Proxmox node goes down, the application cluster (e.g., Postgres, Redis, Kubernetes nodes) handles the failover independently.

## 💾 Storage & Container Runtime
* **TrueNAS Scale:** Acts as the home base for all local data and internal Docker Compose applications. 
* **ZFS for Apps:** Every application gets its own ZFS dataset. This allows for application-specific snapshots, quick rollbacks, and independent offsite replication.
* **Docker vs. Kubernetes Strategy:** * **Local/Internal apps:** Deployed via Docker Compose directly on TrueNAS.
    * **Public-facing apps:** Orchestrated via Kubernetes in the remote colocation servers.

---

## 🛠️ The Software Stack: 50+ Self-Hosted Services

### 1. Dashboards & Documentation
* **Homepage:** A YAML-configured, highly customizable starting page linking out to all services, stripped of heavy API checks to keep it fast.
* **LittleLink / Quick Links:** An environment variable-configured link server page.
* **Jekyll (Chirpy Theme):** A static site generator that turns markdown notes into a beautiful, public-facing documentation site.
* **Shlink:** Advanced custom URL shortener built on a modern architecture (Postgres & Redis backend).

### 2. Core Infrastructure & Routing
* **Traefik:** The primary reverse proxy routing 68+ internal routes and handling SSL certificate termination. Also acts as an ingress for Kubernetes.
* **Pi-Hole:** Runs locally and remotely for ad-blocking and local DNS (CNAME/A records). 
* **Nebula Sync:** Syncs local DNS configurations across all three Pi-Hole instances.
* **Unbound:** A recursive DNS resolver.
* **Keepalived:** Provides a Virtual IP (VIP) load balancer to failover primary DNS nodes smoothly, preventing painful application timeouts when a primary DNS server updates or restarts.

### 3. Monitoring, Logging, & Metrics
* **Prometheus & Grafana:** Prometheus scrapes metrics (from natively supported apps or via Exporters) and stores them in a time-series database. Grafana visually maps this data into actionable dashboards (UPS battery life, GPU stats, TrueNAS performance).
* **Loki & Alloy:** Alloy scrapes application logs and sends them to Loki, which natively integrates into Grafana for powerful standard-out/error log searching.
* **Dazzle:** A lightweight real-time log viewer to quickly troubleshoot flapping containers without using SSH.
* **Uptime Kuma & Uptime Robot:** Kuma handles internal service ping monitoring (using a MySQL backend for stability), while Robot checks external visibility from outside the network.
* **Scrutiny & nvtop:** Scrutiny aggregates hard drive SMART data into a nice dashboard; `nvtop` runs in a container to track GPU utilization (crucial for Plex transcoding).

### 4. Media & Entertainment
* **Plex & Jellyfin:** Plex serves as the primary media library, while Jellyfin runs concurrently pointed at the same datasets as a fail-safe backup.
* **Tautulli:** Deep analytics and watch-history tracking for Plex.
* **Kometa:** Metadata and collection automation. It can scrape Trakt.tv lists and automatically build custom categorized movie collections inside Plex.
* **Handbrake:** Batch transcoding and video conversion.
* **HDHomeRun & Ersatz:** HDHomeRun captures live over-the-air TV via antenna. Ersatz allows you to create custom 24/7 virtual TV channels out of local media (like recreating a retro MTV channel).
* **Dispatchar:** Aggregates channels and converts M3U playlists to XML feeds, bypassing Plex's limitation of only allowing one live TV source type.

### 5. Document Management & Visuals
* **Paperless-ngx:** The central repository for digitizing physical documents. Combined with **Paperless AI** for powerful vision-model extraction, **Tika** for text extraction, and **Gotenberg** to convert HTML/Office docs to PDF.
* **Excalidraw & Draw.io:** Excalidraw handles quick, whiteboard-style squiggly diagrams. Draw.io handles highly structured rigid network topography maps.
* **Racula:** Dedicated server rack diagramming to keep physical inventory and layout documented.

### 6. Workflows & Social
* **n8n:** A deeply powerful, node-based workflow automation tool (think local IFTTT/Zapier on steroids) that ties API actions together across different services.
* **Postiz:** Cross-platform social media scheduling and automation for content creators or marketing teams.

### 7. Home Automation & IoT
* **Home Assistant:** The central brain of the smart home.
* **MQTT & Zigbee2MQTT:** A local message bus that enables diverse Zigbee smart devices to talk locally to Home Assistant without needing proprietary vendor hubs.
* **Scrypted:** A camera bridge/NVR that takes UniFi Protect camera feeds and forces them into Apple HomeKit natively.

### 8. Databases & Administration
* **PostgreSQL & MariaDB:** The primary relational databases backing the majority of the apps.
* **Valkey:** A high-performance, open-source Redis-compatible in-memory data store.
* **Adminer:** A highly lightweight, simple web UI for managing MariaDB/MySQL databases easily.
* **DBGate:** An advanced web SQL client built structurally similar to VS Code.

### 9. AI & Developer Tools
* **Ollama & Open WebUI:** Ollama acts as the local LLM host, and Open WebUI provides the ChatGPT-style web interface to interact with those models.
* **Code Server:** A browser-based version of VS Code with direct dataset mounts to TrueNAS. It allows for quick YAML/Compose file editing without needing to SSH.
* **IT-Tools:** A self-hosted "junk drawer" of incredibly useful dev utilities (JWT token parsers, Base64 decoders, hash generators) that keeps sensitive data off public websites.
* **OpenSpeedTest:** A local network speed tester to validate internal connection speeds (e.g., verifying a 2.5Gbit link between a workstation and the server).

### 10. Kubernetes & GitOps (Colocation)
* **Rancher:** The management UI for 3 remote K3s/RKE clusters.
* **Flux & Renovate:** Flux enforces Infrastructure-as-Code (GitOps). Renovate tracks container dependencies and automatically opens pull requests on a local git repo when a container needs updating. 
* **GitLab Runners:** Performs local CI/CD pipelines to build/deploy code automatically upon commit.
* **MinIO:** Self-hosted S3-compatible object storage (though potentially migrating to RustFS due to licensing).

### 11. Booting & Power Management
* **netboot.xyz:** Allows any machine on the network to boot OS installers directly from the network/web without needing a physical bootable USB drive.
* **NUT (Network UPS Tools) & Peanut:** NUT monitors Uninterruptible Power Supplies and coordinates safe shutdowns during outages. Peanut provides a modern Web UI for NUT and pushes battery metrics into Prometheus.
