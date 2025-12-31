# Chatbot Package

## Quick Start

### 1. Install Docker Desktop
Download: https://www.docker.com/products/docker-desktop/

### 2. Load Images (one-time)
```bash
docker load -i clarity_v1.tar
docker load -i nibbles_v1.tar
```

### 3. Run
```bash
docker-compose up
```

### 4. Open in Browser
- Clarity (Staff): http://localhost:5000
- Nibbles (Customer): http://localhost:5001

### 5. Stop
Press `Ctrl+C`, then:
```bash
docker-compose down
```

## Files in This Package

| File | Purpose |
|------|---------|
| clarity_v1.tar | Docker image for Clarity |
| nibbles_v1.tar | Docker image for Nibbles |
| docker-compose.yml | Runs both chatbots |
| llm.env | API keys for LLM |
| Team_Cashew_Synthetic_Data/ | CSV data files |
