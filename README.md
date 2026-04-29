# 🤖 AgriBot - Smart Farming Systems

AgriBot is a premium, AI-powered agricultural platform designed to support farmers with real-time crop diagnosis, intelligent tutoring, and knowledge testing. It leverages advanced Large Language Models (LLMs) and Vision AI to provide high-accuracy solutions for modern farming challenges.

## 🌟 Key Features
- **AI Assistant**: Real-time crop diagnosis via image upload or text description with AI-detected urgency levels (High, Medium, Low).
- **Quiz Bot**: Dynamic MCQs in Text, Image, and Audio formats to test and improve agricultural knowledge.
- **Tutor Bot**: A conversational AI tutor powered by a local knowledge base (RAG) for factual, agriculture-specific answers.
- **Progress Tracking**: Personal dashboard with visual charts to monitor learning performance over time.
- **Multilingual Support**: Supports both English and Hindi for accessibility.

## 🛠️ Tech Stack
- **Backend**: FastAPI (Python), SQLite (Database), FAISS (Vector Search).
- **AI Models**: Groq API (Llama 3.1 & Llama 4 Scout) for reasoning and vision analysis.
- **Frontend**: Vanilla HTML5, CSS3 (Glassmorphism design), and JavaScript.
- **DevOps**: Docker, Kubernetes (MicroK8s), GitHub Actions, AWS EC2.

---

## 🚀 Deployment Guide (AWS EC2 + Docker)

### 1. Prerequisites
- AWS Account with an active EC2 instance (Ubuntu 22.04+, t2.medium or higher recommended).
- Docker installed on the EC2 instance.
- Your `.pem` key pair for SSH access.
- Groq API Key.

### 2. AWS Security Group Configuration
Ensure the following **Inbound Rules** are set on your EC2 Security Group:

| Type       | Protocol | Port | Source    |
|------------|----------|------|-----------|
| SSH        | TCP      | 22   | 0.0.0.0/0 |
| HTTP       | TCP      | 80   | 0.0.0.0/0 |
| HTTPS      | TCP      | 443  | 0.0.0.0/0 |

### 3. Connecting to Your EC2 Instance

#### Option A — SSH from local terminal (Windows PowerShell)
```bash
# Fix key permissions first (Windows)
icacls "C:\path\to\Agribot.pem" /inheritance:r /grant:r "$($env:USERNAME):(R)"

# SSH into the instance
ssh -i "C:\path\to\Agribot.pem" ubuntu@<EC2-PUBLIC-IP>
```

#### Option B — EC2 Instance Connect (No PEM required) ✅ Recommended
1. Go to **AWS Console → EC2 → Instances**
2. Select your instance → Click **"Connect"**
3. Choose **"EC2 Instance Connect"** tab → Click **"Connect"**

### 4. Build & Run with Docker (Recommended)

Clone the repository on your EC2 instance and build the Docker image:

```bash
# Clone the repo (if not already present)
git clone https://github.com/your-username/AgriBot---Smart-Farming-Systems.git
cd AgriBot---Smart-Farming-Systems

# Build the Docker image
sudo docker build -t agribot .

# Run the container on port 80
sudo docker run -d -p 80:8000 \
  -e GROQ_API_KEY=your_groq_api_key \
  -e SECRET_KEY=your_jwt_secret_key \
  agribot
```

Access the app at: **http://\<EC2-PUBLIC-IP\>**

### 5. Managing the Container

```bash
# Check running containers
sudo docker ps

# View live logs
sudo docker logs -f <container_name>

# Stop the container
sudo docker stop <container_name>

# Restart the container
sudo docker restart <container_name>

# Auto-restart on EC2 reboot
sudo docker update --restart always <container_name>
```

### 6. Updating the App (Hot Patch — No Rebuild)

To update individual files inside a running container without rebuilding:

```bash
# Copy updated file into the running container
sudo docker cp ./backend/database.py <container_name>:/app/backend/database.py

# Restart the container to apply changes
sudo docker restart <container_name>
```

---

## ⚙️ Kubernetes Deployment (MicroK8s — Optional)

MicroK8s is pre-installed on the EC2 instance. To deploy via Kubernetes:

### 1. Create API Key Secrets
```bash
microk8s kubectl create secret generic agribot-secrets \
    --from-literal=groq-api-key=YOUR_GROQ_KEY \
    --from-literal=secret-key=YOUR_JWT_SECRET
```

### 2. Apply the Manifests
```bash
microk8s kubectl apply -f k8s/agribot.yaml
```

### 3. Check Pod Status
```bash
# List all pods
microk8s kubectl get pods

# Check services
microk8s kubectl get services

# View pod logs
microk8s kubectl logs <pod-name>

# Describe pod (for debugging)
microk8s kubectl describe pod <pod-name>
```

> **Note:** Since MicroK8s `LoadBalancer` doesn't auto-assign external IPs on bare EC2, use port-forwarding:
> ```bash
> microk8s kubectl port-forward service/agribot-service 80:80 --address 0.0.0.0 &
> ```

---

## 🔁 CI/CD with GitHub Actions

1. Go to your GitHub Repository **Settings → Secrets and variables → Actions**.
2. Add the following secrets:
   - `DOCKERHUB_USERNAME`: Your Docker Hub username.
   - `DOCKERHUB_TOKEN`: Your Docker Hub personal access token.
   - `KUBE_CONFIG`: The contents of your `~/.kube/config` file.
   - `GROQ_API_KEY`: Your Groq API key.

Every push to the `main` branch will automatically build the Docker image and deploy it.

---

## 🐛 Known Issues & Fixes

### 1. `bcrypt` / `passlib` Incompatibility (500 on Login)

**Problem:** `bcrypt >= 4.0` raises `ValueError: password cannot be longer than 72 bytes` inside `passlib`'s internal `detect_wrap_bug()` function, causing 500 errors on all login/signup requests.

**Fix Applied:**
- Pinned `bcrypt==3.2.2` in `requirements.txt`.
- Updated `database.py` to encode and truncate passwords to 72 bytes before hashing/verifying:

```python
def hash_password(password: str) -> str:
    # bcrypt has a 72-byte max limit — truncate to avoid ValueError
    return pwd_context.hash(password.encode("utf-8")[:72])

def verify_password(plain_password: str, hashed_password: str) -> bool:
    # bcrypt has a 72-byte max limit — truncate to avoid ValueError
    return pwd_context.verify(plain_password.encode("utf-8")[:72], hashed_password)
```

**To apply inside a running container (without rebuild):**
```bash
sudo docker exec <container_name> pip install "bcrypt==3.2.2"
sudo docker restart <container_name>
```

### 2. `pip3` Not Available on Ubuntu 24.04

**Problem:** Ubuntu 24.04 uses an externally-managed Python environment — `pip3` is blocked.

**Fix:** Use Docker (recommended) or a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

---

## 📦 Project Structure
```text
AgriBot---Smart-Farming-Systems/
├── backend/
│   ├── app.py              # FastAPI routes & API logic
│   ├── database.py         # SQLite DB & bcrypt password handling
│   ├── auth.py             # JWT token generation & verification
│   ├── quiz_logic.py       # AI quiz generation & evaluation (RAG)
│   └── media_handler.py    # Image, audio & video processing
├── frontend/               # HTML, CSS, & JavaScript UI
├── k8s/
│   └── agribot.yaml        # Kubernetes Deployment & Service manifest
├── .github/workflows/      # GitHub Actions CI/CD Pipeline
├── data/                   # Knowledge base & vector index files
├── uploads/                # User-uploaded media files
├── Dockerfile              # Container build configuration
├── requirements.txt        # Python dependencies (bcrypt==3.2.2 pinned)
└── README.md               # Documentation
```

## 📝 License
This project is licensed under the MIT License.
