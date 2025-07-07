# 🌍 Wanderlust Travel Blog (MERN + Docker)

Welcome to **Wanderlust**, a full-stack travel blog project built using the **MERN stack** (MongoDB, Express, React, Node.js) and fully containerized using **Docker**.

This guide covers everything from setup to running the application using Docker Compose.

---

## 📂 Project Structure

```bash
wanderlust/
├── backend/              # Express API server
│   ├── Dockerfile
│   ├── .env.sample
│   └── ...
├── frontend/             # React frontend (Vite)
│   ├── Dockerfile
│   ├── .env.sample
│   └── ...
├── docker-compose.yml    # Multi-container setup
└── README.md             # You are here!
```

---

## ⚙️ Phase 1: Dockerizing the Application

### ✅ MongoDB (Database)

**Image Used**: `mongo:latest`  
**Port**: `27017`  
**Volume**:
```yaml
volumes:
  - ./backend/data:/data
```

This persists the MongoDB data between restarts using a bind mount.

---

### ✅ Backend (Node.js + Express)

**Directory**: `./backend`  
**Port**: `5000`  
**Dockerfile**:
```Dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000

CMD ["npm", "start"]
```

**Important Notes**:
- Backend connects to MongoDB using the container name `mongo`:
  ```
  mongodb://mongo/wanderlust
  ```

---

### ✅ Frontend (React + Vite)

**Directory**: `./frontend`  
**Port**: `5173`  
**Dockerfile**:
```Dockerfile
FROM node:18

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host"]
```

**Environment File** (`.env.sample`):
```env
VITE_API_PATH=http://backend:5000
```

This allows the frontend to reach the backend via Docker's internal service name `backend`.

---

## 🔁 Phase 2: Docker Compose Setup

`docker-compose.yml`:

```yaml
version: '3.8'
services:
  mongodb:
    container_name: mongo
    image: mongo:latest
    ports:
      - "27017:27017"
    volumes:
      - ./backend/data:/data

  backend:
    container_name: backend
    build: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - mongodb
    env_file:
      - ./backend/.env.sample

  frontend:
    container_name: frontend
    build: ./frontend
    ports:
      - "5173:5173"
    depends_on:
      - backend
    env_file:
      - ./frontend/.env.sample
```

---

## 🛠️ Phase 3: Running the App

Make sure Docker and Docker Compose are installed.

### Build & Start:
```bash
docker-compose up --build
```

### Stop:
```bash
docker-compose down
```

---

## 🗂️ Phase 4: Import Sample Data

You can import sample posts into MongoDB using:

```bash
docker exec -it mongo mongoimport   --db wanderlust   --collection posts   --file /data/sample_posts.json   --jsonArray
```

> This command loads initial blog post data into MongoDB.

---

## 🌐 Phase 5: Accessing the App

- **Frontend**: http://localhost:5173
- **Backend API**: http://localhost:5000/api/posts

Replace `localhost` with your IP if accessing externally.

---

## ✅ Final Checklist

- [x] MongoDB running and accepting connections.
- [x] Backend properly linked to MongoDB.
- [x] Frontend fetching data from backend using `VITE_API_PATH`.
- [x] Sample posts imported and visible in UI.

---

## 📌 To Do (Optional Enhancements)

- Add Redis & caching
- Implement user authentication
- Deploy to cloud (Render, Heroku, EC2 with Nginx, etc.)
- CI/CD pipeline using GitHub Actions
