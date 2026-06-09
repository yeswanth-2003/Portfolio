# Chepuri Yeswanth — Portfolio
### Dockerized & deployed via GitHub Actions CI/CD · Hosted on Vercel

---

## Overview

This is my personal developer portfolio built with plain **HTML, CSS, and JavaScript** — no frameworks, no build tools, just clean code served through an **Nginx web server** inside a **Docker container**.

When I push code to GitHub, a **GitHub Actions** workflow automatically builds a new Docker image and pushes it to Docker Hub. The live site is hosted on **Vercel**, which auto-deploys on every push to `main` independently.

---

## Tech Used

| Layer | Tool |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JS |
| Web Server | Nginx (Alpine) |
| Containerization | Docker |
| CI/CD | GitHub Actions |
| Image Registry | Docker Hub |
| Hosting | Vercel |

---

## Project Structure

```
portfolio/
├── index.html                     # The portfolio website
├── Dockerfile                     # Builds the Docker image
└── .github/
    └── workflows/
        └── deploy.yml             # GitHub Actions pipeline
```

---

## How the Pipeline Works

```
git push → GitHub Actions triggers → Docker image built → pushed to Docker Hub
                                                        → Vercel auto-deploys live site
```

1. You push any change to the `main` branch
2. GitHub Actions spins up a runner and checks out your code
3. It builds a Docker image using your `Dockerfile`
4. It logs into Docker Hub using your saved secrets
5. It pushes the image to Docker Hub — available to pull on any machine
6. Vercel simultaneously detects the push and redeploys the live site automatically

---

## Why No SSH Auto-Deploy to EC2?

The `deploy.yml` workflow intentionally **does not SSH into an EC2 instance** to redeploy. Here's why:

- This portfolio is a **static site** — Vercel handles hosting for free, forever, with zero configuration and instant global CDN. There's no reason to pay for or manage a server for a static file.
- EC2 is billed by uptime. Running a `t2.micro` 24/7 just to serve a static HTML file wastes free tier hours that are better used for real backend projects.
- The Docker image on Docker Hub serves as a **portable, self-contained artifact**. Anyone (or any server) can pull and run it instantly without touching the source code.
- Keeping CI/CD and hosting concerns separate makes the pipeline simpler and easier to maintain.

The Docker + GitHub Actions setup here is about **demonstrating DevOps skills** — the ability to containerize an app and automate image delivery — not about replacing a perfectly good static host.

---

## Build It Yourself

### Prerequisites
- [Docker](https://www.docker.com/products/docker-desktop/) installed
- A [GitHub](https://github.com) account
- A [Docker Hub](https://hub.docker.com) account

---

### Step 1 — Clone or create the project

```bash
mkdir portfolio
cd portfolio
```

Place your `index.html` inside this folder.

---

### Step 2 — Create the Dockerfile

Create a file named `Dockerfile` (no extension) with this content:

```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

This tells Docker to:
- Use the official lightweight Nginx Alpine image (~23MB vs ~140MB for full nginx)
- Copy your HTML file into the web server's serving directory
- Expose port 80 so traffic can reach it
- Start Nginx in the foreground (required for Docker containers)

---

### Step 3 — Test it locally

```bash
# Build the image
docker build -t my-portfolio .

# Run it
docker run -p 8080:80 my-portfolio
```

Open `http://localhost:8080` in your browser — your portfolio should be live.

---

### Step 4 — Push the project to GitHub

```bash
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/YOUR_USERNAME/portfolio.git
git push -u origin main
```

---

### Step 5 — Create a Docker Hub access token

1. Log into [hub.docker.com](https://hub.docker.com)
2. Go to **Account Settings → Security → New Access Token**
3. Give it a name like `github-actions` and copy the token

> Use an access token instead of your password — it can be revoked anytime without changing your account password.

---

### Step 6 — Add secrets to GitHub

Go to your GitHub repo → **Settings → Secrets and variables → Actions → New repository secret**

Add these two secrets:

| Name | Value |
|---|---|
| `DOCKER_HUB_USERNAME` | Your Docker Hub username |
| `DOCKER_HUB_TOKEN` | The access token you just created |

---

### Step 7 — Create the GitHub Actions workflow

Create the file `.github/workflows/deploy.yml`:

```yaml
name: Deploy Portfolio

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t my-portfolio .

      - name: Log in to Docker Hub
        run: echo "${{ secrets.DOCKER_HUB_TOKEN }}" | docker login -u "${{ secrets.DOCKER_HUB_USERNAME }}" --password-stdin

      - name: Push image to Docker Hub
        run: |
          docker tag my-portfolio ${{ secrets.DOCKER_HUB_USERNAME }}/my-portfolio:latest
          docker push ${{ secrets.DOCKER_HUB_USERNAME }}/my-portfolio:latest
```

---

### Step 8 — Push and watch it run

```bash
git add .
git commit -m "add docker and ci/cd"
git push origin main
```

Go to your repo → **Actions tab** — you'll see the workflow running. Once it's green, your image is live on Docker Hub.

---

### Step 9 — Pull and run the image anywhere

On any machine with Docker installed:

```bash
docker run -p 8080:80 YOUR_DOCKERHUB_USERNAME/my-portfolio:latest
```

---

## Optional: Auto-Deploy to AWS EC2 via SSH

If you want GitHub Actions to also **SSH into an EC2 instance and redeploy automatically** on every push, follow these additional steps.

> **When to use this:** You have a long-running EC2 instance (e.g. for a backend/MERN app) and want zero-downtime redeployments without manually SSHing in every time.

### Step 1 — Add EC2 secrets to GitHub

| Secret Name | Value |
|---|---|
| `EC2_HOST` | Your EC2 public IP address |
| `EC2_USER` | `ubuntu` (default for Ubuntu AMIs) |
| `EC2_SSH_KEY` | Full contents of your `.pem` key file |

To get your `.pem` contents:
```bash
cat your-key.pem
# Copy everything including -----BEGIN RSA PRIVATE KEY----- lines
```

### Step 2 — Make sure Docker is installed on EC2

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ubuntu
```

### Step 3 — Update deploy.yml to add the SSH step

Add this job after the existing `deploy` job in your `deploy.yml`:

```yaml
  redeploy-ec2:
    runs-on: ubuntu-latest
    needs: deploy   # waits for image to be pushed first

    steps:
      - name: SSH into EC2 and redeploy
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKER_HUB_USERNAME }}/my-portfolio:latest
            docker stop portfolio || true
            docker rm portfolio || true
            docker run -d --name portfolio -p 8080:80 ${{ secrets.DOCKER_HUB_USERNAME }}/my-portfolio:latest
```

### How it works

```
git push
    ↓
Build & push image to Docker Hub   (job 1)
    ↓
SSH into EC2                        (job 2, runs after job 1)
    ↓
Pull latest image
    ↓
Stop & remove old container
    ↓
Start new container → site updated ✅
```

> **Note:** The `|| true` after `docker stop` and `docker rm` prevents the pipeline from failing if no container is running yet (e.g. first deploy).

---

## Summary

| Feature | Tool | Cost |
|---|---|---|
| Source control | GitHub | Free |
| Container image | Docker Hub | Free |
| CI/CD pipeline | GitHub Actions | Free (2000 min/month) |
| Live hosting | Vercel | Free forever |
| Optional server deploy | AWS EC2 | Free tier / paid |

---

*Built by Chepuri Yeswanth · MERN Stack Developer*
