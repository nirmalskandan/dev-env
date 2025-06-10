### 🔗 Repository: `dev-env`

#### 📦 Description
Combines `frontend`, `backend`, and `MongoDB` using Docker Compose for a local development or production environment.

> Assumes `frontend`, `backend`, and `dev-env` are sibling folders in a parent `app/` directory.

#### 🐳 Docker Compose Setup
```bash
git clone https://github.com/nirmalskandan/dev-env.git
cd dev-env

# Development mode (includes hot reload and volumes mounted)
docker compose --profile dev up --build

# Production mode (optimized containers without hot reload)
docker compose --profile prod up --build
```

#### 🛠️ Docker Compose File Assumptions
In `dev-env/docker-compose.yml`, relative paths are used:
```yaml
services:
  backend:
    build:
      context: ../backend
      dockerfile: Dockerfile.prod

  frontend:
    build:
      context: ../frontend
      dockerfile: Dockerfile.dev

  mongo:
    image: mongo:6.0
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db
```

Make sure Docker has access to the `../frontend`, `../backend`, and `mongo-data` volumes.

#### 🧭 Access URLs
- Frontend: http://localhost:3000
- Backend API Docs: http://localhost:8000/docs
- MongoDB UI (if connected via a UI client): `mongodb://localhost:27017`

#### 🧪 Test MongoDB Connection
Visit: `http://localhost:3000/mongo-test` — to see MongoDB info retrieved by the backend.

## ✅ Local MongoDB Integration Setup for Frontend, Backend, and Dev-Env

This guide will help you configure your **FastAPI backend** and **React frontend** to connect to a **locally installed MongoDB** running at `localhost:27017`.

---

### 🧱 Assumptions

* You are using **Windows** OS
* MongoDB Compass is available (for GUI access)
* The backend and frontend are already cloned and structured as:

```
project-root/
├── backend/
├── frontend/
└── dev-env/
```

---

### 🍃 Install MongoDB Locally (Windows)

#### 1. ✅ Download MongoDB

* Go to: [https://www.mongodb.com/try/download/community](https://www.mongodb.com/try/download/community)
* Choose: **Windows** → **MSI installer** → Download

#### 2. ✅ Install MongoDB

* Run the installer
* **Choose Complete setup**
* Select "Install MongoDB as a Service" (recommended)
* Continue with default options and finish the setup

#### 3. ✅ Create DB Directory (if service not used)

```powershell
mkdir C:\data\db
```

#### 4. ✅ Start MongoDB Manually (if not a service)

```powershell
cd "C:\Program Files\MongoDB\Server\6.0\bin"
.\mongod.exe --dbpath "C:\data\db"
```

#### 5. ✅ Test Connection in Another Terminal

```powershell
cd "C:\Program Files\MongoDB\Server\6.0\bin"
.\mongo.exe
```

If it connects without error, MongoDB is working.

#### 6. ✅ Optional: Use MongoDB Compass

Open MongoDB Compass and connect to:

```
mongodb://localhost:27017
```

Continue to set up your backend and frontend to connect to MongoDB using the respective local ports.

---

Let me know if you’d like these steps saved in a separate Markdown file or converted into a script!
