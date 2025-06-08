### 🔗 Repository: `dev-env`

#### 📦 Description
Combines `frontend` and `backend` using Docker Compose for a local dev or production environment.

> Assumes `frontend` and `backend` folders are siblings of `dev-env`.

#### 🐳 Docker Compose Setup
```bash
git clone https://github.com/nirmalskandan/dev-env.git
cd dev-env
docker compose --profile dev up --build  # For development
# or
docker compose --profile prod up --build  # For production
```

#### 🛠️ Docker Compose File Assumptions
In `dev-env/docker-compose.yml`, use relative paths:
```yaml
services:
  backend:
    build:
      context: ../backend
    ...

  frontend:
    build:
      context: ../frontend
    ...
```
Make sure Docker has access to those folders.

#### 🧭 Access URLs
- Frontend: http://localhost:3000
- Backend API docs: http://localhost:8000/docs
