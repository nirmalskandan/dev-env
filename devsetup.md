# 🛠️ Step-by-Step: Restructuring Repos for Modular Docker Setup

## 📂️ Repo Structure (No Rename Needed)

| Repo Path   | Purpose                                        |
| ----------- | ---------------------------------------------- |
| `backend/`  | Application backend (FastAPI)                  |
| `frontend/` | Application frontend (React + TypeScript)      |
| `dev-env/`  | Infrastructure: MongoDB, Redis, Docker network |

---

## ✅ Step-by-Step Migration Plan

### 1. **Open WSL and Create Shared Docker Network**

Always run Docker commands inside WSL (or a Linux terminal) if you're on Windows with WSL2:

```bash
# Open WSL terminal
wsl

# Create shared Docker network once (skip if already created)
docker network create app-shared
```

This allows all containers to communicate across repos.

---

### 2. **Ensure All Repos Are Git-Tracked and Isolated**

* Confirm that `backend/`, `frontend/`, and `dev-env/` are individual Git repositories (or git subfolders)
* No folder renaming needed
* Each folder should have its own `.gitignore`, `.env.template`, and `README.md`

---

### 3. **Update **\`\`** to Handle Infra and Profiles**

```yaml
services:
  mongo:
    image: mongo:6.0
    container_name: mongo
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db
    environment:
      MONGO_INITDB_DATABASE: mydatabase
    profiles:
      - dev
      - prod

  redis:
    image: redis:7.2
    container_name: redis
    restart: always
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    profiles:
      - dev
      - prod

volumes:
  mongo_data:
  redis_data:

networks:
  default:
    external: true
    name: app-shared
```

To run only infra:

```bash
cd dev-env
# Open WSL terminal before running
wsl
# Start dev profile only
docker compose --profile dev up -d
```

---

### 4. **Update **\`\`** to Use Shared Network and Profiles**

```yaml
services:
  backend:
    container_name: backend
    build:
      context: .
      dockerfile: Dockerfile.dev
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: uvicorn main:app --host 0.0.0.0 --port 8000 --reload
    environment:
      - MONGO_URL=mongodb://mongo:27017/mydatabase
      - REDIS_URL=redis://redis:6379
    profiles:
      - dev

  backend-prod:
    container_name: backend-prod
    build:
      context: .
      dockerfile: Dockerfile.prod
    ports:
      - "8000:8000"
    command: uvicorn app.main:app --host 0.0.0.0 --port 8000
    environment:
      - MONGO_URL=mongodb://mongo:27017/mydatabase
      - REDIS_URL=redis://redis:6379
    profiles:
      - prod

networks:
  default:
    external: true
    name: app-shared
```

Run from WSL:

```bash
cd backend
wsl
docker compose --profile dev up -d
```

---

### 5. **Update **\`\`** to Use Shared Network and Profiles**

```yaml
services:
  frontend:
    container_name: frontend
    build:
      context: .
      dockerfile: Dockerfile.dev
    volumes:
      - .:/app
      - /app/node_modules
    ports:
      - "3000:3000"
    command: npm run dev -- --host
    profiles:
      - dev

  frontend-prod:
    container_name: frontend-prod
    build:
      context: .
      dockerfile: Dockerfile.prod
    ports:
      - "3000:80"
    command: ["nginx", "-g", "daemon off;"]
    profiles:
      - prod

networks:
  default:
    external: true
    name: app-shared
```

Run from WSL:

```bash
cd frontend
wsl
docker compose --profile dev up -d
```

---

### 6. **Update React Frontend to Use Env Variable**

1. Create a file named `.env.local` at the root of the `frontend/` repo:

```env
VITE_API_URL=http://backend:8000
```

2. In your React app, update all API calls to use this variable:

```ts
fetch(`${import.meta.env.VITE_API_URL}/your-endpoint`)
```

3. Make sure Vite is configured to expose the variable. Vite automatically loads any `.env.*` files.

4. If you're already running the dev server, stop and restart it:

```bash
# Inside frontend/
wsl
docker compose down
docker compose --profile dev up -d
```

This ensures that the `.env.local` values are loaded into the container.

---

### 7. **CI/CD for Backend (GitHub Actions)**

Use ephemeral containers in the workflow:

```yaml
services:
  mongo:
    image: mongo
    ports:
      - 27017:27017

  redis:
    image: redis
    ports:
      - 6379:6379
```

Include DB connection in `env:` section.

---

### 8. **CI/CD for Frontend (GitHub Actions)**

Option A: Use mock/stub API.

Option B: Spin up both `backend` and `frontend` Docker Compose for integration testing.

---

### 9. **Optional: Local Dev Bootstrap Script**

Create a file `run-local.sh` at `project-root/`:

```bash
#!/bin/bash
# Open WSL terminal first, then run this script

(cd dev-env && docker compose --profile dev up -d)
(cd backend && docker compose --profile dev up -d)
(cd frontend && docker compose --profile dev up -d)
```

Make it executable:

```bash
chmod +x run-local.sh
```

---

### 10. **Documentation**

Each repo should include:

* `README.md` with Docker + `.env` setup
* `.env.template`
* Section for `app-shared` Docker network usage

---

### 11. **Optional: Local Dev Shutdown Script**

Create a file `stop-local.sh` at `project-root/`:

```bash
#!/bin/bash
# Open WSL terminal first, then run this script

(cd frontend && docker compose down)
(cd backend && docker compose down)
(cd dev-env && docker compose down)
```

Make it executable:

```bash
chmod +x stop-local.sh
```

To manually stop a container with a conflicting name:

```bash
docker rm -f backend
```

Or list all containers and remove as needed:

```bash
docker ps -a
docker rm -f <container_name>
```

---

## 🧯 Common Errors & Fixes

### ❌ Error: "Error connecting to backend"

* **Cause**: React can't resolve `http://backend:8000`.
* **Fix**: Add `.env.local` with `VITE_API_URL=http://backend:8000` and restart dev server.
* **Also Check**: Is your backend actually running inside Docker with container name `backend`?

### ❌ Error: `net::ERR_NAME_NOT_RESOLVED` in React

* **Cause**: React can't resolve the hostname `backend`.
* **Fix**: Ensure `frontend` and `backend` are on the same Docker network (`app-shared`).

### ❌ Error: `curl` not found inside container

* **Fix**: Use Alpine-friendly install: `apk add curl` (if you're inside Alpine)

### ❌ Error: Backend not going down

* **Fix**:

  ```bash
  docker ps -a | grep backend
  docker rm -f backend
  ```

### ❌ Still not working?

* **Try**:

  ```bash
  docker network ls
  docker inspect app-shared
  ```

  Make sure all containers are connected.

---

## 🎹️ Comparison to Monolithic Compose Setup (from Canva 1)

| Criteria         | Modular Docker (Current)        | Monolithic Docker Compose           |
| ---------------- | ------------------------------- | ----------------------------------- |
| Dev Modularity   | ✅ Each service in separate repo | ❌ All services coupled              |
| CI Safety        | ✅ Ephemeral containers          | ❌ Risk of shared infra interference |
| Scalability      | ✅ Easy to scale teams/repos     | ❌ One central setup to maintain     |
| Production-alike | ✅ Closer to prod reality        | ⚠️ Artificially centralized         |
| Complexity       | ⚠️ Slightly more setup needed   | ✅ Easy to get started               |

**Modular Docker strategy was chosen** because it encourages:

* Closer match to production setups
* Clean separation of concern
* CI/CD flexibility
* Team autonomy and scaling

The extra effort leads to better collaboration and long-term maintainability.
