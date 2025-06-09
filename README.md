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