## ✅ Updated VM Deployment guide for cashew chatbots

---

````markdown
# 🖥️ VM Deployment Guide (Docker, TAR-based)

This guide explains how to deploy the **Clarity** and **Nibbles** chatbots onto a cloud VM using **prebuilt Docker images (`.tar`)** and **Docker Compose**.

The instructions are written for **Ubuntu VMs** and assume Docker images are built locally and uploaded to the VM.

---

## 🎯 End Result

| Chatbot | URL |
|--------|-----|
| Clarity | `http://YOUR_VM_IP:5000` |
| Nibbles | `http://YOUR_VM_IP:5001` |

---

## 📋 Prerequisites

- Ubuntu VM with public IP
- SSH access
- Docker + Docker Compose installed
- Ports **5000** and **5001** open in:
  - VM firewall (ufw)
  - Cloud provider NSG / security group

---

## 📁 Expected VM Directory Layout

All chatbot assets live under:

```text
/mnt/data/cashew-chatbot
├── Team_Cashew_Synthetic_Data/
│   ├── sku_master.csv
│   ├── sales_transactions.csv
│   └── (other CSVs)
├── templates/
│   ├── clarity.html
│   ├── nibbles.html
│   └── logo.png
├── llm.env
├── docker-compose.yml
├── clarity_v1.tar
└── nibbles_v1.tar
````

---

## 🔧 Step 1: SSH into the VM

```bash
ssh ubuntu@YOUR_VM_IP
```

---

## 🐳 Step 2: Install Docker (Ubuntu)

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker $USER
exit
```

SSH back in:

```bash
ssh ubuntu@YOUR_VM_IP
```

Verify installation:

```bash
docker --version
docker compose version
```

---

## 📁 Step 3: Create Project Folder on VM

```bash
sudo mkdir -p /mnt/data/cashew-chatbot
sudo chown -R $USER:$USER /mnt/data/cashew-chatbot
cd /mnt/data/cashew-chatbot
```

---

## 📤 Step 4: Upload Files from Local Machine

From your **local machine**, upload:

* `clarity_v1.tar`
* `nibbles_v1.tar`
* `docker-compose.yml`
* `llm.env`
* `templates/`
* `Team_Cashew_Synthetic_Data/`

Example using `scp`:

```bash
scp clarity_v1.tar nibbles_v1.tar docker-compose.yml llm.env \
    ubuntu@YOUR_VM_IP:/mnt/data/cashew-chatbot/

scp -r templates Team_Cashew_Synthetic_Data \
    ubuntu@YOUR_VM_IP:/mnt/data/cashew-chatbot/
```

---

## 📦 Step 5: Load Docker Images on VM

```bash
cd /mnt/data/cashew-chatbot
docker load -i nibbles_v1.tar
docker load -i clarity_v1.tar
```

Verify:

```bash
docker images | egrep -i 'nibbles|clarity'
```

---

## 📄 Step 6: docker-compose.yml (Final)

Ensure your `docker-compose.yml` looks like this:

```yaml
services:
  nibbles:
    image: nibbles:v1
    container_name: nibbles
    ports:
      - "5001:5001"
    env_file:
      - /mnt/data/cashew-chatbot/llm.env
    environment:
      - PORT=5001
      - ENV_FILE=/app/llm.env
      - DATA_DIR=/app/Team_Cashew_Synthetic_Data
      - COMPANY_NAME=Cashew4Nuts
      - CURRENCY=SGD
    volumes:
      - /mnt/data/cashew-chatbot/Team_Cashew_Synthetic_Data:/app/Team_Cashew_Synthetic_Data:ro
      - /mnt/data/cashew-chatbot/templates:/app/templates:ro
      - /mnt/data/cashew-chatbot/llm.env:/app/llm.env:ro
    restart: unless-stopped

  clarity:
    image: clarity:v1
    container_name: clarity
    ports:
      - "5000:5000"
    env_file:
      - /mnt/data/cashew-chatbot/llm.env
    environment:
      - PORT=5000
      - ENV_FILE=/app/llm.env
      - DATA_DIR=/app/Team_Cashew_Synthetic_Data
    volumes:
      - /mnt/data/cashew-chatbot/Team_Cashew_Synthetic_Data:/app/Team_Cashew_Synthetic_Data:ro
      - /mnt/data/cashew-chatbot/templates:/app/templates:ro
      - /mnt/data/cashew-chatbot/llm.env:/app/llm.env:ro
    restart: unless-stopped
```

---

## 🚀 Step 7: Start the Chatbots

```bash
docker compose up -d
docker compose ps
```

---

## 🔓 Step 8: Open Firewall Ports

```bash
sudo ufw allow 5000
sudo ufw allow 5001
sudo ufw status
```

Ensure your cloud provider firewall also allows inbound TCP on **5000** and **5001**.

---

## ✅ Step 9: Test the Chatbots

```bash
curl http://localhost:5000/health
curl http://localhost:5001/healthz
```

From browser:

* `http://YOUR_VM_IP:5000` → **Clarity**
* `http://YOUR_VM_IP:5001` → **Nibbles**

---

## 🔍 Useful Docker Commands

```bash
docker compose ps
docker logs -n 100 clarity
docker logs -n 100 nibbles
docker compose restart
docker compose down
```

---

## ❓ Troubleshooting

| Issue                     | Fix                                |
| ------------------------- | ---------------------------------- |
| Page loads but no replies | Check `llm.env`                    |
| CSV not found             | Check `DATA_DIR` mount             |
| Logo missing              | Ensure `templates/logo.png` exists |
| Port unreachable          | Check VM + cloud firewall          |
| Container restarting      | `docker logs <container>`          |

```
Just say 👍
```
