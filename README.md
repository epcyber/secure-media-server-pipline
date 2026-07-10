# Private Media Server & Cloud Backup Setup

This repository documents how I built a secure, private photo and media server out of a home computer (an OptiPlex 5040 running Xubuntu Linux). 

Instead of relying on public cloud applications, this setup keeps 100% of the data completely under personal control. It features isolated application spaces, a private encrypted network tunnel, a smart local backup vault, and an automated cloud disaster recovery pipeline.

---

## 🏗️ The Application Stack

Here is a simple look at how the system layers fit together:

1. **Immich (Docker):** The core application where media is uploaded, managed, and viewed.
2. **Tailscale:** A private security shield that allows secure logins from anywhere in the world without exposing open ports to the public internet.
3. **Vorta & BorgBackup:** A local backup engine that takes daily automated snapshots, shrinking them down so they do not waste local storage.
4. **Cloud Script (Rclone):** An automated compression pipeline that packs everything up, uploads a fresh mirror to the cloud, and enforces a 48-hour rolling cleanup to keep storage costs at zero.

---

## 🚀 Step-by-Step Deployment Guide & Configuration

### Step 1: Run the App Safely Using Docker
I used **Docker** to run the media server inside an isolated box. This prevents application files from mixing up or breaking the underlying Linux operating system.

1. Create a dedicated folder on the server for the application:
   ```bash
   mkdir ~/immich-app && cd ~/immich-app
   ```
2. Download the configuration blueprints (`docker-compose.yml` and `.env` files) provided by the application developers.
   ```bash
   wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
   wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
   ```
3. Open the environment configuration file to map storage locations and database passwords:
   ```bash
   nano .env
   ```
Start the application stack in the background:
   ```bash
   docker compose up -d
   ```

### 🔒 Step 2: Lock Down the Network with Tailscale
Normally, to access a home server from outside your house, you have to open ports on your home router, leaving it open can possibly cause a breach in the home network. I bypassed this entirely using a private meshnet via Tailscale.
1. Install Tailscale on the server using the native setup script:
   ```bash
   curl -fsSL https://tailscale.com/install.sh | sh
   ```
2. Start the service and authenticate the server into your secure, private meshnet pool:
   ```bash
   sudo tailscale up
   ```
3. Install the Tailscale client on your phone or laptop, log into the same account, and use the assigned private IP address to securely manage your server from anywhere.

### 💾 Step 3: Set Up Smart Local Backups (Vorta & Borg)
Backing up a media server by copying and pasting identical folders every day wastes massive amounts of drive space. I used BorgBackup (managed by a visual interface called Vorta).

Borg uses block-level deduplication. If you have a 5 GB video file, Borg saves it the first day. On the second day, if that video has not changed, Borg skips it entirely and only saves the brand-new data variations, preventing local storage bloat.

1. Install Vorta on your Linux desktop environment:
   ```bash
   sudo apt install vorta
   ```
2. Open Vorta, navigate to the Repository tab, and initialize a new local backup vault on your dedicated backup drive.
3. Go to the Sources tab and select your raw immich data directory.
4. Set a rolling automation schedule under the Schedule tab to trigger a snapshot every night at 2:00 AM using a Grandfather-Father-Son (GFS) retention schedule.

## ☁️ Step 4: Automate a Fast Cloud Fallback Copy
If your server's local drives fail physically or your house loses power, local backups won't save your data. You need a secondary fallback copy in the cloud.

To fix this, I combined two tools to create an efficient pipeline:

__1. Bundle and Shrink Everything at Once__: I used a tool called tar to pack thousands of files into one big single archive. To avoid letting the backup take hours, I tell tar to use a helper tool called pigz. Instead of forcing just one part of your computer's processor to do all the heavy lifting, pigz unleashes every single available core of your CPU to compress your data blocks at the exact same time.

__2. Upload It Using Your Full Internet Speed__: I used a tool called rclone to send the final file to my cloud storage account. I configured it to move data in large, optimized 128-megabyte chunks. This structure makes sure the system fully utilizes the high-speed internet connection, pushing the upload bandwidth to its limit so the transfer finishes as fast as possible.

__3. Keep Storage Clean on Autopilot__: I added a cleanup rule at the absolute end of the script (--min-age 2d). This automatically tells the cloud account to look for and delete any backup file older than 48 hours. This locks the cloud footprint to exactly two rolling files, preventing data bloat and keeping the storage costs minimal.

## 🛠️ Deployment:
1. Install the cloud transport engine and the multi-core compression helper:
   ```bash
   sudo apt install rclone pigz
   ```
2. Link your personal cloud account (like Google Drive, OneDrive, or Azure Blob) by running the configuration wizard:
   ```bash
   rclone config
   ```
3. Create a simple automated task sheet (a shell script) to combine everything on autopilot:
   ```bash
   nano ~/backup.sh
   ```
