### 🔗 Repository: `dev-env`

#### 📦 Description
Combines `frontend` and `backend` using Docker Compose for a local dev or production environment.

#### 🐳 Docker Compose Setup
```bash
git clone https://github.com/nirmalskandan/dev-env.git
cd dev-env
git clone https://github.com/nirmalskandan/backend.git
cd backend && cd ..
git clone https://github.com/nirmalskandan/frontend.git
cd frontend && cd ..
```

#### 🛠️ Run in Development Mode
```bash
docker compose --profile dev up --build
```
- Uses live reload, mounted volumes, and dev tools.

#### 🚀 Run in Production Mode
```bash
docker compose --profile prod up --build
```
- Uses production builds with no live reload.

#### 🧭 Access URLs
- Frontend: http://localhost:3000
- Backend API docs: http://localhost:8000/docs