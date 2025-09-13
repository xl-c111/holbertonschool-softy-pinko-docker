# holbertonschool-softy-pinko-docker

In **Task 6**, we extend the previous setup (Task 5) by **running multiple back-end API servers** and using **Nginx as a proxy/load balancer**. This allows traffic to be distributed across multiple API servers using the **Round-Robin algorithm**, improving scalability and reliability.

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
└── docker-compose.yml
```

---

## ⚙️ Setup / Prerequisites

- Install [Docker](https://docs.docker.com/get-docker/)
- Install [Docker Compose](https://docs.docker.com/compose/install/)
- Clone this repository or copy the `task6/` directory

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

### Step 3 – Open in browser

- Front-end: [http://localhost/](http://localhost/)
- Back-end API (through proxy): [http://localhost/api/hello](http://localhost/api/hello)

### Step 4 – Verify Load Balancing

Reload `http://localhost/api/hello` multiple times.  
In your terminal logs you should see alternating requests handled by **back-end-1** and **back-end-2**.

✅ With this setup:

- **Front-end** serves the static UI.
- **Back-end** serves API responses.
- **Proxy** unifies them at `http://localhost`.
- **Docker Compose** manages everything, and scaling is as simple as `--scale back-end=2`.

---

## ⚙️ File-by-File Explanation

### 1. **back-end/**

- **Dockerfile**
  - Builds an image based on `ubuntu:latest`.
  - Installs Python3 and Flask.
  - Installs `flask-cors` to handle cross-origin requests.
  - Copies `api.py` into the container.
  - Runs the Flask server on **port 5252**.

- **api.py**
  - A simple Flask app exposing an endpoint like:
    ```python
    @app.route("/api/hello")
    def hello():
        return "Hello, World!"
    ```
  - Responds with `"Hello, World!"` to test communication between front-end and back-end.

- **requirements.txt**
  - Lists Python dependencies (`flask`, `flask-cors`).
  - Ensures the build installs the correct versions consistently.

---

### 2. **front-end/**

- **Dockerfile**
  - Uses the `nginx:latest` image.
  - Copies static front-end files (HTML, CSS, JS) into `/var/www/html/softy-pinko-front-end`.
  - Copies the Nginx config `softy-pinko-front-end.conf` to serve those files.
  - Exposes **port 9000** internally for other containers.

- **softy-pinko-front-end/**
  - Contains the Softy Pinko template (index.html, CSS, JS, assets).
  - Displays the **Hello, World! landing page**.

- **softy-pinko-front-end.conf**
  - Configures Nginx to serve files from `/var/www/html/softy-pinko-front-end` on port **9000**.

---

### 3. **proxy/**

- **Dockerfile**
  - Uses the `nginx:latest` image.
  - Copies `proxy.conf` to `/etc/nginx/conf.d/default.conf`.
  - Runs Nginx as the **reverse proxy** and load balancer.

- **proxy.conf**  
  Configures Nginx to:

  ```nginx
  server {
      listen 80;

      # Route root requests to front-end
      location / {
          proxy_pass http://front-end:9000;
      }

      # Route /api requests to back-end (load balanced across multiple)
      location /api {
          proxy_pass http://back-end:5252;
      }
  }
  ```

  - `/` → goes to the **front-end container** (port 9000).
  - `/api` → goes to the **back-end containers** (port 5252).
  - Nginx automatically balances traffic between **back-end-1** and **back-end-2**.

---

### 4. **docker-compose.yml**

Defines and connects all services:

- **back-end**
  - Builds from `./back-end/Dockerfile`.
  - Creates **multiple containers** when scaled.
  - Exposes port **5252** internally (not mapped to host).

- **front-end**
  - Builds from `./front-end/Dockerfile`.
  - Serves the static site on port **9000** (internal only).
  - Depends on back-end.

- **proxy**
  - Builds from `./proxy/Dockerfile`.
  - Listens on **host port 80** and forwards requests.
  - Depends on front-end and back-end.

---

## 🔄 How It All Works Together

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
