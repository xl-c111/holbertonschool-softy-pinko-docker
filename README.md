# Task 6 – Horizontal Scaling with Docker Compose and Nginx Proxy

In **Task 6**, we extend the previous setup (Task 5) by **running multiple back-end API servers** and using **Nginx as a proxy/load balancer**. This allows traffic to be distributed across multiple API servers using the **Round-Robin algorithm**, improving scalability and reliability.

---

## ⚙️ Setup / Prerequisites

- Install [Docker](https://docs.docker.com/get-docker/)
- Install [Docker Compose](https://docs.docker.com/compose/install/)
- Clone this repository or copy the `task6/` directory

All dependencies (Flask, Flask-CORS, Nginx) are installed automatically inside Docker containers during the build process.

---

## 📂 Project Structure

```
task6/
│── back-end/
│   ├── Dockerfile
│   ├── api.py
│   └── requirements.txt
│
│── front-end/
│   ├── Dockerfile
│   ├── softy-pinko-front-end/   # static HTML, CSS, JS
│   └── softy-pinko-front-end.conf
│
│── proxy/
│   ├── Dockerfile
│   └── proxy.conf
│
│── docker-compose.yml
│── 2-api-servers.txt   # contains the command to run 2 API servers
```

---

## ▶️ How to Run Task 6

### Step 1 – Build all services

```bash
docker-compose build
```

### Step 2 – Start with 2 back-end containers

```bash
docker-compose up --scale back-end=2
```

This command is also saved in `2-api-servers.txt` (required by the checker).

### Step 3 – Open in browser

- Front-end: [http://localhost/](http://localhost/)
- Back-end API (through proxy): [http://localhost/api/hello](http://localhost/api/hello)

### Step 4 – Verify Load Balancing

Reload `http://localhost/api/hello` multiple times.  
In your terminal logs you should see alternating requests handled by **back-end-1** and **back-end-2**.

---

## 🔄 How It Works

### ASCII Flow Diagram

```
          +------------------+
          |    Browser       |
          |  http://localhost|
          +--------+---------+
                   |
                   v
          +------------------+
          |     Proxy        |
          |  Nginx (port 80) |
          +---+----------+---+
              |          |
   /----------+          +-----------/api
        v                            v
+-------------------+       +-------------------+
|   Front-end       |       |   Back-end-1      |
| Nginx (port 9000) |       | Flask (port 5252) |
+-------------------+       +-------------------+
                              ^
                              |
                              v
                      +-------------------+
                      |   Back-end-2      |
                      | Flask (port 5252) |
                      +-------------------+
```

- Requests to `/` → sent to **Front-end (Nginx)** → serves static website.
- Requests to `/api` → load-balanced between **Back-end-1** and **Back-end-2**.

### Explanation

- **Browser** sends requests to `http://localhost`.
- **Proxy (Nginx)** listens on port 80 and routes traffic:
  - `/` → goes to the **front-end container** (Nginx, port 9000).
  - `/api` → load-balanced between **back-end containers** (Flask, port 5252).
- **Front-end** serves static HTML, CSS, and JS.
- **Back-end** responds with API data (`Hello, World!`).
- **Docker Compose** orchestrates all containers and networking.

---

## ⚙️ File-by-File Explanation

### 1. **back-end/**

- **Dockerfile**
  - Builds from `ubuntu:latest`.
  - Installs Python3, pip3, Flask, and Flask-CORS.
  - Copies `api.py`.
  - Runs Flask server on port **5252**.

- **api.py**
  - Simple Flask app with an `/api/hello` endpoint returning `"Hello, World!"`.

- **requirements.txt**
  - Defines Python dependencies (Flask, Flask-CORS).

---

### 2. **front-end/**

- **Dockerfile**
  - Uses `nginx:latest`.
  - Copies static site into `/var/www/html/softy-pinko-front-end`.
  - Uses `softy-pinko-front-end.conf` for configuration.
  - Exposes port **9000** internally.

- **softy-pinko-front-end/**
  - Static files (HTML, CSS, JS).

- **softy-pinko-front-end.conf**
  - Configures Nginx to serve static files from `/var/www/html/softy-pinko-front-end`.

---

### 3. **proxy/**

- **Dockerfile**
  - Uses `nginx:latest`.
  - Copies `proxy.conf` → `/etc/nginx/conf.d/default.conf`.

- **proxy.conf**  
  Configures routing:

  ```nginx
  server {
      listen 80;

      # Route root requests to front-end
      location / {
          proxy_pass http://front-end:9000;
      }

      # Route /api requests to back-end
      location /api {
          proxy_pass http://back-end:5252;
      }
  }
  ```

---

### 4. **docker-compose.yml**

Defines services:

- **back-end**
  - Build from `./back-end/Dockerfile`.
  - Can be scaled (`--scale back-end=2`).
  - Exposes **5252** internally.

- **front-end**
  - Build from `./front-end/Dockerfile`.
  - Serves static site on port **9000** internally.

- **proxy**
  - Build from `./proxy/Dockerfile`.
  - Listens on **host port 80**.
  - Routes to front-end and back-end.

---

## ✅ Summary

- **Front-end** → serves UI.
- **Back-end** → serves API.
- **Proxy** → unifies access and load-balances requests.
- **Docker Compose** → manages containers, scaling, and networking.

Scaling horizontally is achieved by running multiple back-end containers with:

```bash
docker-compose up --scale back-end=2
```
