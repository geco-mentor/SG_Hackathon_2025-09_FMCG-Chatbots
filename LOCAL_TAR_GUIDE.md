# 🐳 Local Docker Distribution Guide

Share the chatbots with colleagues for local proof-of-concept demos.

---

## 📁 What You (The Builder) Need

Your folder:

```
chatbot-project\
├── Dockerfile.clarity
├── Dockerfile.nibbles
├── .dockerignore
├── Clarity_patch_v0_0_1.py
├── Nibbles_patch_v0_0_1.py
├── llm.env
└── Team_Cashew_Synthetic_Data\
    ├── sku_master.csv
    └── (other CSVs)
```

---

## 🔨 Step 1: Build Docker Images

```bash
cd chatbot-project

docker build -f Dockerfile.clarity -t clarity:v1 .
docker build -f Dockerfile.nibbles -t nibbles:v1 .
```

---

## 💾 Step 2: Export as .tar

```bash
docker save clarity:v1 -o clarity_v1.tar
docker save nibbles:v1 -o nibbles_v1.tar
```

---

## 📦 Step 3: Create the Package for Colleagues

Folder to share:

```
chatbot-package\
├── clarity_v1.tar
├── nibbles_v1.tar
├── docker-compose.yml
├── llm.env
└── Team_Cashew_Synthetic_Data\
    ├── sku_master.csv
    └── (other CSVs)
```

---

## 📄 docker-compose.yml (include this in package)

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
```

---

## 📋 Instructions for Colleagues

### One-Time Setup

1. Install Docker Desktop: https://www.docker.com/products/docker-desktop/
2. Start Docker Desktop (wait for "Running")
3. Open terminal, go to the package folder:
   ```bash
   cd path\to\chatbot-package
   ```
4. Load the images:
   ```bash
   docker load -i clarity_v1.tar
   docker load -i nibbles_v1.tar
   ```

### Run

```bash
docker-compose up
```

### Access

- Clarity: http://localhost:5000
- Nibbles: http://localhost:5001

### Stop

`Ctrl+C`, then:
```bash
docker-compose down
```

---

## 🔄 Updates

| Changed | Action |
|---------|--------|
| Python script | Rebuild image, export new .tar, reshare |
| API keys | Update llm.env, reshare |
| CSV data | Update folder, reshare |
