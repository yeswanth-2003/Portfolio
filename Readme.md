# Chepuri Yeswanth — Portfolio
### Dockerized & deployed via GitHub Actions CI/CD

---

## How I Built This

This is my personal developer portfolio built with plain **HTML, CSS, and JavaScript** — no frameworks, no build tools, just clean code served through an **Nginx web server** inside a **Docker container**.

When I push code to GitHub, a **GitHub Actions** workflow automatically builds a new Docker image and pushes it to Docker Hub. That's the entire pipeline — simple, fast, and fully automated.

---

## Tech Used

| Layer | Tool |
|---|---|
| Frontend | HTML5, CSS3, Vanilla JS |
| Web Server | Nginx (Alpine) |
| Containerization | Docker |
| CI/CD | GitHub Actions |
| Image Registry | Docker Hub |

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
```

1. You push any change to the `main` branch
2. GitHub Actions spins up a runner and checks out your code
3. It builds a Docker image using your `Dockerfile`
4. It logs into Docker Hub using your saved secrets
5. It pushes the image — done

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
- Use the official lightweight Nginx image
- Copy your HTML file into the web server's root
- Serve it on port 80

---

### Step 3 — Test it locally

```bash
# Build the image
docker build -t my-portfolio .

# Run it
docker run -p 80:80 my-portfolio
```

Open `http://localhost` in your browser — your portfolio should be live.

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
docker run -p 80:80 YOUR_DOCKERHUB_USERNAME/my-portfolio:latest
```

---

## That's It

No complex infrastructure. No paid services. Just Docker + GitHub Actions — and your portfolio is containerized and automatically deployed every time you push a change.

---

*Built by Chepuri Yeswanth · MERN Stack Developer*