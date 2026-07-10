# Private Media Server & Cloud Backup Setup

This repository documents how I built a secure, private photo and media server out of a home computer (an OptiPlex 5040 running Xubuntu Linux). 

Instead of relying on public cloud applications, this setup keeps 100% of the data completely under personal control. It features isolated application spaces, a private encrypted network tunnel, a smart local deduplicated backup vault, and an automated cloud disaster recovery pipeline.

---

## 🏗️ The Application Stack

Here is a simple look at how the system layers fit together:

1. **Immich (Docker):** The core application where media is uploaded, managed, and viewed.
2. **Tailscale:** A private security shield that allows secure logins from anywhere in the world without exposing open ports to the public internet.
3. **Vorta & BorgBackup:** A local backup engine that takes daily automated snapshots, shrinking them down so they do not waste local storage.
4. **Cloud Script (Rclone):** An automated compression pipeline that packs everything up, blasts a fresh mirror to the cloud, and enforces a strict 48-hour rolling cleanup to keep storage costs at zero.

---

## 🚀 Step-by-Step Deployment Guide & Configuration

### Step 1: Run the App Safely Using Docker
We use **Docker** to run the media server inside an isolated box. This prevents application files from mixing up or breaking the underlying Linux operating system.

1. Create a dedicated folder on the server for the application:
   ```bash
   mkdir ~/immich-app && cd ~/immich-app
