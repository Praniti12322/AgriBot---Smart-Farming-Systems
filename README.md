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
- **DevOps**: Docker, Kubernetes, GitHub Actions, AWS (EC2/EKS).

---

## 🚀 Deployment Guide (AWS + Kubernetes)

### 1. Prerequisites
- AWS Account with CLI configured.
- Docker Hub account.
- GitHub Repository for your project.

### 2. Infrastructure Setup (AWS CLI)

#### Step 1: Create an EC2 Instance (for Docker-only deployment)
If you just want to run a Docker container on a single instance:
```bash
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --count 1 \
    --instance-type t2.medium \
    --key-name MyKeyPair \
    --security-groups AgriBotSG
```

#### Step 2: Create an EKS Cluster (for Kubernetes deployment)
```bash
eksctl create cluster \
    --name agribot-cluster \
    --region us-east-1 \
    --nodegroup-name standard-nodes \
    --node-type t3.medium \
    --nodes 2
```

### 3. Dockerization
To build and run the container locally:
```bash
# Build the image
docker build -t agribot:latest .

# Run the container
docker run -p 8000:8000 --env-file .env agribot:latest
```

### 4. Kubernetes Deployment
1. Create a secret for your API keys:
```bash
kubectl create secret generic agribot-secrets \
    --from-literal=groq-api-key=YOUR_GROQ_KEY \
    --from-literal=secret-key=YOUR_JWT_SECRET
```

2. Apply the manifests:
```bash
kubectl apply -f k8s/agribot.yaml
```

### 5. CI/CD with GitHub Actions
1. Go to your GitHub Repository **Settings > Secrets and variables > Actions**.
2. Add the following secrets:
   - `DOCKERHUB_USERNAME`: Your Docker Hub username.
   - `DOCKERHUB_TOKEN`: Your Docker Hub personal access token.
   - `KUBE_CONFIG`: The contents of your `~/.kube/config` file (required for K8s deployment).
   - `GROQ_API_KEY`: Your Groq API key.

Now, every time you push to the `main` branch, the pipeline will automatically build the image and deploy it to your cluster.

---

## 📦 Project Structure
```text
integrated_project/
├── backend/            # FastAPI logic & AI integration
├── frontend/           # HTML, CSS, & JavaScript
├── k8s/               # Kubernetes manifests
├── .github/workflows/ # CI/CD Pipeline
├── Dockerfile          # Container configuration
├── requirements.txt    # Python dependencies
└── README.md           # Documentation
```

## 📝 License
This project is licensed under the MIT License.
