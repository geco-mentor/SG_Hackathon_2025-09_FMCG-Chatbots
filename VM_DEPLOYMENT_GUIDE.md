# 🖥️ VM Deployment Guide

Deploy chatbots on your Bitdeer VM so anyone can access via URL. Guide was originally written for deployment on Bitdeer's cloud, but the concepts should be universally applicable.

---

## 🎯 End Result

| Chatbot | URL |
|---------|-----|
| Clarity | `http://YOUR_VM_IP:5000` |
| Nibbles | `http://YOUR_VM_IP:5001` |

Or with a domain (optional):
| Chatbot | URL |
|---------|-----|
| Clarity | `http://clarity.yourdomain.com` |
| Nibbles | `http://nibbles.yourdomain.com` |

---

## 📋 Prerequisites

- SSH access to your Bitdeer VM
- VM's public IP address
- Ports 5000 and 5001 open (or 80/443 if using domain)

---

## 🔧 Step 1: SSH into Your VM

```bash
ssh username@YOUR_VM_IP
```

Or use PuTTY on Windows.

---

## 🐳 Step 2: Install Docker on VM

### For Ubuntu/Debian:

```bash
# Update packages
sudo apt update

# Install Docker
sudo apt install -y docker.io docker-compose

# Start Docker
sudo systemctl start docker
sudo systemctl enable docker

# Add your user to docker group (so you don't need sudo)
sudo usermod -aG docker $USER

# Log out and back in for group change to take effect
exit
```

SSH back in:
```bash
ssh username@YOUR_VM_IP
```

Verify Docker works:
```bash
docker --version
docker-compose --version
```

---

## 📁 Step 3: Create Project Folder on VM

```bash
mkdir -p ~/chatbot-project
cd ~/chatbot-project
```

---

## 📤 Step 4: Upload Files to VM

From your **local machine** (not the VM), upload the files:

### Option A: Using SCP (Mac/Linux/Windows PowerShell)

```bash
# Upload .tar files
scp clarity_v1.tar username@YOUR_VM_IP:~/chatbot-project/
scp nibbles_v1.tar username@YOUR_VM_IP:~/chatbot-project/

# Upload llm.env
scp llm.env username@YOUR_VM_IP:~/chatbot-project/

# Upload data folder
scp -r Team_Cashew_Synthetic_Data username@YOUR_VM_IP:~/chatbot-project/
```

### Option B: Using FileZilla (GUI)

1. Download FileZilla: https://filezilla-project.org/
2. Connect: Host=`YOUR_VM_IP`, Username, Password, Port=22
3. Navigate to `/home/username/chatbot-project/` on right side
4. Drag and drop files from left side

---

## 📄 Step 5: Create docker-compose.yml on VM

SSH into VM:
```bash
ssh username@YOUR_VM_IP
cd ~/chatbot-project
```

Create the file:
```bash
nano docker-compose.yml
```

Paste this content:

```yaml
version: '3.8'

services:
  clarity:
    image: clarity:v1
    container_name: clarity-chatbot
    ports:
      - "5000:8080"
    environment:
      - PORT=8080
      - DATA_DIR=/app/data
    env_file:
      - llm.env
    volumes:
      - ./Team_Cashew_Synthetic_Data:/app/data:ro
    restart: unless-stopped

  nibbles:
    image: nibbles:v1
    container_name: nibbles-chatbot
    ports:
      - "5001:8080"
    environment:
      - PORT=8080
      - DATA_DIR=/app/data
    env_file:
      - llm.env
    volumes:
      - ./Team_Cashew_Synthetic_Data:/app/data:ro
    restart: unless-stopped
```

Save: `Ctrl+O`, Enter, `Ctrl+X`

---

## 📦 Step 6: Load Docker Images

```bash
cd ~/chatbot-project

docker load -i clarity_v1.tar
docker load -i nibbles_v1.tar
```

Verify:
```bash
docker images
```

---

## 🚀 Step 7: Start the Chatbots

```bash
docker-compose up -d
```

The `-d` flag runs in background (detached mode).

Check they're running:
```bash
docker ps
```

You should see both containers running.

---

## 🔓 Step 8: Open Firewall Ports

### On the VM itself:

```bash
# Ubuntu/Debian with ufw
sudo ufw allow 5000
sudo ufw allow 5001
sudo ufw status
```

### On Bitdeer's Cloud Console:

1. Log into Bitdeer's cloud management console
2. Find your VM's security group / firewall rules
3. Add inbound rules:
   - Port 5000, TCP, from 0.0.0.0/0 (or your IP range)
   - Port 5001, TCP, from 0.0.0.0/0 (or your IP range)

---

## ✅ Step 9: Test the URLs

Share these with your team:

| Chatbot | URL |
|---------|-----|
| **Clarity** | `http://YOUR_VM_IP:5000` |
| **Nibbles** | `http://YOUR_VM_IP:5001` |

Replace `YOUR_VM_IP` with your actual VM IP (e.g., `http://203.45.67.89:5000`)

---

## 🔍 Useful Commands

```bash
# Check running containers
docker ps

# View logs
docker logs clarity-chatbot
docker logs nibbles-chatbot

# Follow logs in real-time
docker logs -f clarity-chatbot

# Stop everything
docker-compose down

# Start everything
docker-compose up -d

# Restart a specific container
docker restart clarity-chatbot
```

---

## 🔄 Updating on VM

### Update API Keys

```bash
cd ~/chatbot-project
nano llm.env          # Edit and save
docker-compose down
docker-compose up -d
```

### Update CSV Data

Upload new files via SCP/FileZilla, then:
```bash
docker-compose down
docker-compose up -d
```

### Update Docker Image

On your local machine, rebuild and export:
```bash
docker build -f Dockerfile.clarity -t clarity:v2 .
docker save clarity:v2 -o clarity_v2.tar
```

Upload to VM:
```bash
scp clarity_v2.tar username@YOUR_VM_IP:~/chatbot-project/
```

On VM:
```bash
cd ~/chatbot-project
docker load -i clarity_v2.tar

# Update docker-compose.yml to use v2
nano docker-compose.yml   # change clarity:v1 to clarity:v2

docker-compose down
docker-compose up -d
```

---

## 🌐 Optional: Nice URLs with Domain Name

If you have a domain (e.g., `yourdomain.com`), you can set up:
- `http://clarity.yourdomain.com`
- `http://nibbles.yourdomain.com`

### Step A: Point DNS to VM

In your domain registrar (GoDaddy, Namecheap, etc.):
- Add A record: `clarity` → `YOUR_VM_IP`
- Add A record: `nibbles` → `YOUR_VM_IP`

### Step B: Install Nginx

```bash
sudo apt install -y nginx
```

### Step C: Configure Nginx

```bash
sudo nano /etc/nginx/sites-available/chatbots
```

Paste:

```nginx
server {
    listen 80;
    server_name clarity.yourdomain.com;

    location / {
        proxy_pass http://localhost:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }
}

server {
    listen 80;
    server_name nibbles.yourdomain.com;

    location / {
        proxy_pass http://localhost:5001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_cache_bypass $http_upgrade;
    }
}
```

Save and enable:

```bash
sudo ln -s /etc/nginx/sites-available/chatbots /etc/nginx/sites-enabled/
sudo nginx -t                    # Test config
sudo systemctl restart nginx
sudo ufw allow 80                # Open HTTP port
```

Now access via:
- `http://clarity.yourdomain.com`
- `http://nibbles.yourdomain.com`

---

## 📁 VM Folder Structure

After setup, your VM should have:

```
~/chatbot-project/
├── clarity_v1.tar
├── nibbles_v1.tar
├── docker-compose.yml
├── llm.env
└── Team_Cashew_Synthetic_Data/
    ├── sku_master.csv
    └── (other CSVs)
```

---

## ❓ Troubleshooting

| Problem | Solution |
|---------|----------|
| Can't access URL | Check firewall (VM + Bitdeer console) |
| "Connection refused" | Container not running — `docker ps` |
| Container keeps restarting | Check logs — `docker logs clarity-chatbot` |
| "sku_master.csv not found" | Check path in docker-compose.yml |
| LLM errors | Check API keys in llm.env |
