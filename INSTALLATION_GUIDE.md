# Installation Guide: Local Setup and First Run

## Objective

This guide installs the core stack for the Personal AI OS using the simplest path: local-first, free-cost, and laptop-friendly.

---

## 1. System Requirements

### Recommended hardware
- 16GB RAM minimum
- 256GB SSD recommended
- CPU with 4+ cores
- Optional NVIDIA GPU for faster local inference

### Software requirements
- Git
- Docker + Docker Compose
- Python 3.10+
- Node.js 18+
- WSL2 recommended for Windows

---

## 2. Install Dependencies

### Linux / macOS

```bash
# Install Git
sudo apt update && sudo apt install git

# Install Docker
curl -fsSL https://get.docker.com | sh

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Install Python
sudo apt install python3 python3-pip python3-venv
```

### Windows (WSL2)

```bash
wsl --install
# then in Ubuntu shell:

sudo apt update
sudo apt install git curl docker.io docker-compose-plugin python3 python3-pip python3-venv nodejs npm
```

---

## 3. Clone the Repo

```bash
git clone https://github.com/soufianeeloirrak9-digital/personal-ai-os.git
cd personal-ai-os
```

---

## 4. Set Up Environment Variables

```bash
cp .env.example .env
```

Then edit `.env` and fill in:
- DATABASE_URL
- REDIS_URL
- OLLAMA_BASE_URL
- Groq / Together / Google keys as needed

---

## 5. Install Ollama

### Linux / macOS

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama serve
```

### Pull recommended models

```bash
ollama pull qwen2.5-coder:8b
ollama pull deepseek-coder:6.7b
ollama pull glm4:9b
```

---

## 6. Install Python Dependencies

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

---

## 7. Install Node Dependencies

```bash
npm install
```

---

## 8. Start the Stack

### Docker-based local stack

```bash
docker-compose up -d
```

### Verify services

```bash
curl http://localhost:3000
curl http://localhost:3001/health
curl http://localhost:11434/api/tags
```

---

## 9. First Run

Open your browser to:
- UI: http://localhost:3000
- API: http://localhost:3001

### Example task

```bash
curl -X POST http://localhost:3001/api/tasks \
  -H "Content-Type: application/json" \
  -d '{
    "task": "Generate a Python function to calculate fibonacci numbers",
    "priority": "medium"
  }'
```

---

## 10. First-Time Troubleshooting

### Ollama not responding
```bash
ollama serve
```

### Docker service failing
```bash
docker-compose logs -f
```

### Model not found
```bash
ollama list
ollama pull qwen2.5-coder:8b
```

### Node install fails
```bash
npm cache clean --force
npm install
```

---

## 11. Production note

This is the local-first setup. For deployment to live infrastructure, use the instructions in `DEPLOYMENT.md`.

---

## 12. Summary

Once set up, the AI OS will run locally and automatically fallback to free cloud APIs when needed. This gives you a low-cost AI operating system that still feels sophisticated and scalable.
