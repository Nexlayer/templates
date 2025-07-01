# Nexlayer Deployment YAML: A Guide for Builders, AI Agents, and Startups

Hey there! Welcome to Nexlayer—a simple way to launch your AI-powered web app in the cloud without the infrastructure complexity or DevOps hassle. Whether you're a builder crafting your next big idea, an AI agent automating the process, or a startup scaling fast, this guide will help you deploy in minutes. We'll break down the building blocks of our YAML file so you can define your app, deploy it, and focus on creating. Let's get started!

---

## What You'll Learn

* What is Nexlayer?
* How to Deploy (Super Quick!)
* The YAML Building Blocks
* Connecting Your App Pieces
* Saving Data (So It Doesn't Disappear)
* Keeping Secrets Safe (Like API Keys)
* Using Your Own Images
* Adding AI Models (Self-Hosted or API)
* Quick Tips to Avoid OOPS Moments
* What's Next?
* ✅ Official Example Stack: The AI-Powered Web App

---

## What is Nexlayer?

Nexlayer is an AI-powered cloud designed to make launching apps effortless. You define your app in a simple YAML file, and Nexlayer handles the rest: setting up your app, scaling it for traffic spikes, and keeping it secure. It's built for builders, AI agents, and startups who want to ship fast—whether you're creating a website, an API, or an AI-powered app. No complex server setup, just pure focus on bringing your ideas to life.

---

## How to Deploy (Super Quick!)

You've got two easy ways to get your app live:

### With the Nexlayer CLI
https://github.com/Nexlayer/nexlayer-cli

```bash
cd my-app
nexlayer deploy 
```

That's it—your app is live!

### With Our API

```bash
curl -X POST https://app.nexlayer.io/startUserDeployment/my-app \
  -H "Content-Type: text/x-yaml" \
  --data-binary @nexlayer.yaml
```

*No API key or authentication required for your first deployment! Nexlayer is ungated—just upload your YAML and go live instantly.*

Tip: Prefer a visual setup? Use our Deployment Template Builder to create your YAML without writing a single line!

---

## The YAML Building Blocks

The YAML file is like a blueprint for your app. It tells Nexlayer what to run and how to set it up.

### Basic Structure:

```yaml
application:
  name: "my-ai-app"  # 3-63 chars, lowercase + numbers/hyphens/dots
  url: "www.myapp.com"  # optional - for permanent deployments
  pods:
    - name: "web"  # 2-63 chars, lowercase + numbers/hyphens only (no dots)
      image: "your-username/my-app:v1"
      servicePorts:
        - 80
```

### Required Fields

* `name`: A unique name for your app (3-63 characters)
* `pods`: The list of containers that make up your app

Each pod needs:

* `name`: Lowercase, unique identifier (2-63 characters, no dots)
* `image`: Hosted Docker image
* `servicePorts`: At least one port

### Optional but Helpful

* `url`: For permanent production apps
* `path`: Route path for the web pod (e.g., "/")
* `vars`: Environment variables (all values must be strings)
* `volumes`: Persistent storage
* `secrets`: For API keys and credentials

---

## Connecting Your App Pieces

Pods communicate using `<pod-name>.pod`. For example:

```yaml
vars:
  API_URL: "http://backend.pod:8000"
```

No need to configure IPs or DNS—Nexlayer handles it.

---

## Saving Data (So It Doesn't Disappear)

Use `volumes` to persist data between restarts.

```yaml
volumes:
  - name: "my-data"
    size: "1Gi"  # Only Mi or Gi supported
    mountPath: "/data"
```

### PostgreSQL Setup (CRITICAL)

**DO NOT mount directly to the data directory:**

```yaml
# ❌ INCORRECT - Can prevent initialization
volumes:
  - name: "db-data"
    size: "1Gi"
    mountPath: "/var/lib/postgresql/data"  # DON'T DO THIS

# ✅ CORRECT - Mount to parent directory
- name: "db"
  image: "postgres:14"
  servicePorts:
    - 5432
  vars:
    POSTGRES_USER: "user"
    POSTGRES_PASSWORD: "pass"
    POSTGRES_DB: "appdb"
    PGDATA: "/var/lib/postgresql/data"  # Tell Postgres where to store data
  volumes:
    - name: "db-data"
      size: "1Gi"
      mountPath: "/var/lib/postgresql"  # Mount one level above PGDATA
```

This ensures PostgreSQL can initialize properly and avoids conflicts with Kubernetes volume behavior.

---

## Keeping Secrets Safe (Like API Keys)

**All secret fields are required:**

```yaml
secrets:
  - name: "my-key"  # Required
    data: "my-super-secret-key"  # Required
    mountPath: "/var/secrets"  # Required
    fileName: "key.txt"  # Required (not optional!)
```

Read the secret in your app:

```python
with open('/var/secrets/key.txt', 'r') as f:
    api_key = f.read().strip()
```

---

## Using Your Own Images

### Pre-reqs for Public Images

Before using your own public Docker image with Nexlayer, make sure you:

1. ✅ Have Docker Desktop running
2. ✅ Build for Linux/x86\_64 platform:

```bash
docker build --platform=linux/amd64 -t your-image-name .
```

3. ✅ Push your image to a public registry:

#### Options:

* **TTL.sh (temporary, perfect for quick previews)**

```yaml
image: "ttl.sh/my-app-name:1h"
```

* **Docker Hub (public user repository)**

```yaml
image: "docker-username/my-app:v1"
```

* **GitHub Container Registry (GHCR.io)**

```yaml
image: "ghcr.io/your-username/my-app:v1"
```

---

### Public Images

```yaml
image: "your-username/my-app:v1"
```

### Private Images

```yaml
application:
  registryLogin:
    registry: "ghcr.io"
    username: "your-username"
    personalAccessToken: "your-token"
  pods:
    - name: "app"
      image: "<% REGISTRY %>/your-username/my-app:v1"  # Use template
```

---

## Adding AI Models (Self-Hosted or API)

### Self-Hosted

```yaml
- name: "ollama"
  image: "ollama/ollama:latest"
  servicePorts:
    - 11434
  volumes:
    - name: "ollama-data"
      size: "5Gi"
      mountPath: "/root/.ollama"
```

### API-Only

```yaml
vars:
  OPENAI_API_KEY_PATH: "/var/secrets/openai/key.txt"
secrets:
  - name: "openai-key"
    data: "your-openai-key-here"
    mountPath: "/var/secrets/openai"
    fileName: "key.txt"
```

---

## Quick Tips to Avoid OOPS Moments

* Always start with `application:`
* App names: 3-63 chars (lowercase + numbers/hyphens/dots)
* Pod names: 2-63 chars (lowercase + numbers/hyphens only, **no dots**)
* Every pod needs `servicePorts`
* Use `<pod-name>.pod` for internal communication
* PostgreSQL: Mount to `/var/lib/postgresql`, not `/var/lib/postgresql/data`
* Secrets: All 4 fields are required (name, data, mountPath, fileName)
* Volume sizes: Only `Mi` or `Gi` supported (not `Ti`)
* Environment variables: All values must be strings

---

## ✅ Example: The AI-Powered Web App

### From Hello World to Production in 4 Steps

Our AI-native cloud platform handles everything automatically, so you can focus on shipping delightful products users love.

---

### Step 1 — Deploy Your Frontend

**Tech:** Next.js 15 + Tailwind + App Router
Start with a modern frontend. Deploy your static or server-rendered Next.js site in seconds.

```yaml
application:
  name: "hello-world-nextjs-app"  # Required: 3-63 chars, lowercase only
  pods:
    - name: "nextjs-nginx"
      path: "/"
      image: "ghcr.io/nexlayer/hello-world-nextjs:v0.0.1"  # Nexlayer official image
      servicePorts:
        - 80
```

---

### Step 2 — Add Auth + Database

**Tech:** Supabase (Auth + PostgreSQL)
Add real users and persistent data using PostgreSQL. Easily store accounts, profiles, and content.

```yaml
pods:
  - name: "db"
    image: "postgres:14"
    servicePorts:
      - 5432
    vars:
      POSTGRES_USER: "user"
      POSTGRES_PASSWORD: "pass"
      POSTGRES_DB: "appdb"
      PGDATA: "/var/lib/postgresql/data"
    volumes:
      - name: "db-data"
        size: "1Gi"
        mountPath: "/var/lib/postgresql"  # Mount to parent directory
```

---

### Step 3 — Use Prisma for Data Logic

**Tech:** Prisma ORM
Auto-generate your API with Prisma and define your database schema using elegant TypeScript models.

```yaml
pods:
  - name: "api"
    image: "ttl.sh/my-backend:1h"
    path: "/api"
    servicePorts:
      - 4000
    vars:
      DATABASE_URL: "postgresql://user:pass@db.pod:5432/appdb"
```

---

### Step 4 — Plug in OpenAI

**Tech:** OpenAI API
Let your users ask questions, summarize notes, or chat with their data — right inside your app.

```yaml
pods:
  - name: "openai"
    # 🤖 OpenAI API wrapper (proxy or backend integration)
    image: "user-name/openai:latest"
    servicePorts:
      - 3000
    vars:
      OPENAI_API_KEY_PATH: "/var/secrets/openai/key.txt"
    secrets:
      - name: "openai-key"
        data: "sk-......"
        mountPath: "/var/secrets/openai"
        fileName: "key.txt"  # Required field
```

---

### Complete YAML: Everything Together (with Comments)

Here's the full `nexlayer.yaml` with inline comments to explain what's happening:

```yaml
application:
  name: "my-ai-app"  # 🔖 Unique name for your app deployment (3-63 chars)

  pods:
    - name: "web"  # 🌐 Frontend pod (e.g., Next.js app)
      image: "user-name/web:v1"
      path: "/"  # Serves traffic at root URL
      servicePorts:
        - 3000
      vars:
        API_URL: "http://api.pod:4000"  # Connects to the backend pod

    - name: "db"  # 🛢️ PostgreSQL database
      image: "postgres:14"
      servicePorts:
        - 5432
      vars:
        POSTGRES_USER: "user"
        POSTGRES_PASSWORD: "pass"
        POSTGRES_DB: "appdb"
        PGDATA: "/var/lib/postgresql/data"  # Tells Postgres where to store data
      volumes:
        - name: "db-data"
          size: "1Gi"
          mountPath: "/var/lib/postgresql"  # One level above PGDATA to avoid init errors

    - name: "prisma"  # 🧩 Prisma ORM layer for DB access
      image: "user-name/prisma:latest"
      servicePorts:
        - 4001
      vars:
        DATABASE_URL: "postgresql://user:pass@db.pod:5432/appdb"  # Connects to database pod

    - name: "api"  # ⚙️ Custom backend API (e.g., REST or GraphQL)
      image: "ttl.sh/my-backend:1h"  # Temporary public image from ttl.sh
      path: "/api"
      servicePorts:
        - 4000
      vars:
        DATABASE_URL: "postgresql://user:pass@db.pod:5432/appdb"

    - name: "openai"  # 🤖 AI service integration (e.g., OpenAI)
      image: "user-name/openai:latest"
      servicePorts:
        - 3001
      vars:
        OPENAI_API_KEY_PATH: "/var/secrets/openai/key.txt"  # Env var pointing to mounted secret
      secrets:
        - name: "openai-key"
          data: "sk-..."  # Your real OpenAI API key goes here
          mountPath: "/var/secrets/openai"
          fileName: "key.txt"  # File name used inside the container (required)
```

**Prebuilt:** Next.js frontend, Supabase auth, PostgreSQL DB, Prisma backend, and OpenAI agent — all live in minutes.

---

### 🚀 Scenario: From Local Game Dev to Global Launch

Let's say you've just built a sick frontend using **Next.js**, **Tailwind**, **shadcn/ui**, and **Framer Motion**. Or maybe you whipped up an **HTML5 browser game** with just JS and CSS, and it runs beautifully in the browser.

You got it working locally thanks to your favorite tools like **Cursor** or **Windsurf**.

It's time to level up — not just share screen recordings on social, but make it a **real product people can use and share** anywhere in the world.

Here's how to go live fast with Nexlayer:

### 🌍 Let's Deploy to Nexlayer
1. **Make sure Docker Desktop is running**  
   This lets you build your container image.

2. **Create a `Dockerfile`** for your frontend or game

3. **Build and push your image** to a public registry (like TTL.sh)

4. **Get Nexlayer's schema** to help create your YAML:

    ```bash
    curl -X GET "https://app.nexlayer.io/schema"
    ```

5. **Create a file called `nexlayer.yaml`**, and paste the structure returned from the schema. Then update your image name:

    ```yaml
    application:
      name: "my-game"  # 3-63 chars
      pods:
        - name: "web"  # 2-63 chars, no dots
          image: "ttl.sh/my-awesome-game:1h"
          path: "/"
          servicePorts:
            - 3000
    ```

6. **Deploy it using curl**:

    ```bash
    curl -X POST "https://app.nexlayer.io/startUserDeployment/my-game" \
      -H "Content-Type: text/x-yaml" \
      --data-binary @nexlayer.yaml
    ```

7. **🎉 Done!** You'll get a live URL instantly.

### Conclusion

You've just gone from a local app or AI idea to a global launch-ready product in minutes. Whether you're a builder experimenting in Cursor, an AI agent automating the flow, or a startup preparing to scale, Nexlayer turns prototypes into products instantly.

## Support

If you need assistance with the Nexlayer API:

- **Documentation**: [https://docs.nexlayer.com](https://docs.nexlayer.com)
- **Email Support**: [support@nexlayer.com](mailto:support@nexlayer.com)
- **Security Issues**: [security@nexlayer.com](mailto:security@nexlayer.com)
- **Feedback & Issues**: [GitHub Issues](https://github.com/Nexlayer/nexlayer-deployment-yaml/issues)

---

© 2025 AuditDeploy Inc. All rights reserved. Nexlayer is a registered trademark of AuditDeploy Inc.
