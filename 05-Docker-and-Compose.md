# Docker & Docker Compose (Weeks 5, 6, 7)

**Docker** = platform to build, package and run apps in lightweight, portable **containers**.
**Image** = read-only template (code + runtime + libs). **Container** = running instance of an image.

---

## A. Docker CLI with Redis (Week 5)

```bash
docker pull redis                          # 1. download image
docker run --name my-redis -d redis        # 2. create+start container in background
docker ps                                  # 3. running containers
docker exec -it my-redis redis-cli         # 4. open redis-cli inside
#   127.0.0.1:6379> SET name "Alice"   -> OK
#   127.0.0.1:6379> GET name           -> "Alice"
#   127.0.0.1:6379> exit
docker stop my-redis                       # 5. stop (not deleted)
docker start my-redis                      # 6. start again
docker stop my-redis
docker rm my-redis                         # 7. delete container (stop first)
docker rmi redis                           # 8. delete image
```

## B. Dockerfile (Week 6)

**Dockerfile** = text file of instructions to build a custom image.

1. Create folder `C:\DockerProjects\Redis` (Windows: `cd /c/DockerProjects/Redis` in Git Bash) or `mkdir -p ~/DockerProjects/Redis && cd ~/DockerProjects/Redis` (Linux/Mac).
2. Create a file named `Dockerfile` (no extension):
```dockerfile
FROM redis:latest
CMD ["redis-server"]
```
3. Commands:
```bash
docker build -t redisnew .                      # build image from Dockerfile in current dir
docker run --name myredisnew -d redisnew        # run
docker ps
docker stop myredisnew
docker login                                    # Docker Hub login
docker ps -a                                    # all containers incl. stopped
docker commit <container_id> <dockerhub-user>/redis1   # snapshot container -> image
docker images
docker push <dockerhub-user>/redis1             # upload to Docker Hub
docker rm <container_id>
docker rmi <dockerhub-user>/redis1
docker ps -a
docker logout
docker pull <dockerhub-user>/redis1             # download it back
docker run --name myredis -d <dockerhub-user>/redis1
docker exec -it myredis redis-cli
#   SET name "Abcdef"  / GET name / exit
docker ps -a
docker stop myredis
docker rm <container_id>
docker images
docker rmi <dockerhub-user>/redis1
docker logout
```

**Common Dockerfile instructions**
| Instruction | Purpose |
|---|---|
| `FROM` | base image |
| `WORKDIR` | working directory inside image |
| `COPY` / `ADD` | copy files into image |
| `RUN` | executes at **build** time (install packages) |
| `CMD` | default command at **run** time (overridable) |
| `ENTRYPOINT` | fixed command at run time |
| `EXPOSE` | documents the port |
| `ENV` | environment variable |

### Docker scenario answers
1. Check running containers: `docker ps`
2. `docker stop web_app`
3. `docker build -t myapi .`
4. `docker run -d --name web_app -p 8080:5000 web_app` (host 8080 → container 5000)
5. `docker exec -it web_app /bin/bash` (use `sh` if bash missing)
6. `docker rmi old_api` (add `-f` to force)
7. `docker logs web_app` (`-f` to follow, `--tail 100` for last lines)
8. `docker run -d <image>` (detached/background)
9. `docker ps --filter "publish=3000"` (or `docker ps | grep 3000`; on host `netstat -ano | findstr 3000`)
10. `docker tag <image_id_or_name> myapp:1.0`
11. Export: `docker save -o myimage.tar myimage:1.0`; import on other machine: `docker load -i myimage.tar`
12. Auto restart: `docker run -d --restart unless-stopped myimage` (or `--restart always` / `on-failure`); for existing: `docker update --restart=always <container>`
13. Usage: `docker stats` (`--no-stream` for a snapshot). Limit: `docker run -d -m 512m --cpus=1 myimage` (`--memory-swap` for swap). Measure actual: `docker stats`, `docker inspect <c>`
14. Run a shell command before the app starts: use an **ENTRYPOINT script** (`COPY entrypoint.sh /`, `ENTRYPOINT ["/entrypoint.sh"]`, script ends with `exec "$@"`) or `CMD ["sh","-c","your-command && python app.py"]`. (`RUN` only runs at build time.)
15. Custom Nginx image:
```dockerfile
FROM nginx:latest
COPY ./static/ /usr/share/nginx/html/
EXPOSE 80
```
```bash
docker build -t my-nginx .
docker run -d -p 8080:80 my-nginx
```

---

## C. Docker Compose (Week 7)

**Compose** defines and runs multi-container apps from one YAML file (`docker-compose.yml`).
Parts: `services` (containers: image, ports, environment, depends_on), `networks`, `volumes`.
(`version:` is optional in newer Compose; harmless to include.)

**Commands**
```bash
docker compose up -d                 # start all (older: docker-compose up -d)
docker compose ps
docker compose logs -f
docker compose down                  # stop+remove (volumes survive)
docker compose down -v               # also delete volumes
docker compose up --scale web=2 -d   # scale a service
docker compose up --build            # rebuild images then start
docker compose restart
```

### Lab 1 — Define multiple interdependent services
```bash
mkdir compose-lab && cd compose-lab
```
`docker-compose.yml`:
```yaml
version: "3.9"
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
```
```bash
docker compose up -d
```
Open http://localhost:8080 → Nginx welcome page; `db` runs in background.

### Lab 2 — Add Redis + depends_on
```yaml
version: "3.9"
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    depends_on:
      - redis
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
  redis:
    image: redis:alpine
```
```bash
docker compose up -d
docker compose ps        # web, db, redis all running
```

### Lab 3 — Deploy on another machine
Zip `compose-lab`, copy to another machine with Docker Compose, then:
```bash
docker compose up -d
```
Same services run without changes.

### Lab 4 — Network + persistent volume
```yaml
version: "3.9"
networks:
  app-net:
volumes:
  db-data:
services:
  web:
    image: nginx:latest
    ports:
      - "8080:80"
    networks:
      - app-net
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - app-net
```
```bash
docker compose up -d
docker compose exec db psql -U demo -d demo_db -c "CREATE TABLE t(id int); INSERT INTO t VALUES (1);"
docker compose down
docker compose up -d
docker compose exec db psql -U demo -d demo_db -c "SELECT * FROM t;"   # data persists
```
Services reach each other by service name (e.g. `db`) over `app-net`.

### Lab 5 — Faster dev iteration (Flask)
`app.py`:
```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Flask + Docker!"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```
`Dockerfile`:
```dockerfile
FROM python:3.10-slim
WORKDIR /app
COPY app.py /app/
RUN pip install flask
CMD ["python", "app.py"]
```
`docker-compose.yml` (web uses `build: .`):
```yaml
version: "3.9"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    depends_on:
      - db
  db:
    image: postgres:15
    environment:
      POSTGRES_USER: demo
      POSTGRES_PASSWORD: demo
      POSTGRES_DB: demo_db
```
```bash
docker compose up --build        # visit http://localhost:5000
# change the return text in app.py, then:
docker compose up --build        # new message appears after rebuild
```

### Compose scenario answers
1. **No** — run one process per container; put Node.js backend and Python scheduler in separate containers (same Compose file) for isolation, independent scaling and easier restarts.
2. Two services in Compose: `flask: ports ["5000:5000"]` and `nginx: ports ["80:80"]`, then `docker compose up -d`.
3. One `docker-compose.yml` with services `frontend` (React), `backend` (Express), `mongo` (MongoDB, volume for data), `depends_on` chain; run with `docker compose up -d`, manage with `docker compose ps/logs/down`.
4. Second container fails: **"port is already allocated"** — a host port can be used by only one container. Map different host ports: `-p 8080:8080` and `-p 8081:8080`.
5. `docker compose restart` (or `docker compose up -d --build` if code changed).
6. `docker compose up -d --build --no-deps frontend` — rebuilds and recreates only the frontend.
7. "Connection refused" on Tomcat: port not published (`-p 8080:8080` missing), container not running/crashed (`docker ps`, `docker logs`), Tomcat still starting, app bound to wrong port, or firewall/security-group blocking.
8. Which container uses port 3000: `docker ps --filter "publish=3000"` or `docker ps | grep 3000`.
9. `docker stop tomcat && docker rm tomcat && docker rmi tomcat` (use your container/image names).
10. Share the image: `docker login` → `docker tag myapp user/myapp:1.0` → `docker push user/myapp:1.0`; teammate runs `docker pull user/myapp:1.0`. Or `docker save -o app.tar myapp` / `docker load -i app.tar`; or share the Compose file + Dockerfile via Git.
