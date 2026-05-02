---
name: docker-essentials
description: Use Docker and Docker Compose from the terminal — run, build, exec, logs, ps, compose up/down, Dockerfile basics, image and volume hygiene. Load when the user wants to run containers, build images, or troubleshoot Docker.
---

# Docker essentials

## When to load this skill

Any task that touches containers — running an image, building one, debugging a running container, working with `docker-compose.yml`, cleaning up disk usage, or troubleshooting "it works in dev but not in container".

## Mental model

- **Image** = read-only filesystem snapshot + metadata (entry point, env, etc.). Built from a `Dockerfile`. Stored in a registry.
- **Container** = a running (or stopped) instance of an image, with its own writable layer + namespaced process tree.
- **Volume** = persistent storage that outlives a container. Mount it where data should survive.
- **Network** = how containers reach each other and the outside world.
- **Registry** = where images live (Docker Hub, GitHub Container Registry, ECR, GCR, …).

## Quick reference

### Run / inspect / stop containers
```sh
docker run hello-world                       # pull + run + exit
docker run -it ubuntu bash                   # interactive shell
docker run -d --name web -p 8080:80 nginx    # detached, named, port-mapped (host:container)
docker run --rm alpine echo hi               # auto-remove when done

docker ps                                    # running containers
docker ps -a                                 # incl. stopped
docker logs <name|id>                        # stdout/stderr
docker logs -f <name>                        # follow
docker logs --tail 100 -f <name>
docker exec -it <name> bash                  # shell into a running container
docker exec <name> ls /etc                   # one-shot command
docker inspect <name>                        # full JSON metadata
docker stats                                 # live CPU/mem
docker top <name>                            # processes inside container

docker stop <name>                           # SIGTERM, then SIGKILL after grace
docker kill <name>                           # ⚠ SIGKILL immediately
docker restart <name>
docker rm <name>                             # remove stopped container
docker rm -f <name>                          # ⚠ stop + remove
```

### Images
```sh
docker images                                # list local images
docker pull alpine:3.20                      # fetch from registry
docker build -t myapp:dev .                  # build from Dockerfile in cwd
docker build -t myapp:dev -f Dockerfile.prod .   # specify Dockerfile
docker tag myapp:dev myapp:1.2.0             # add another tag
docker push ghcr.io/me/myapp:1.2.0           # push to registry (after `docker login`)
docker rmi myapp:dev                         # remove image
docker history myapp:dev                     # see layers
```

### Volumes
```sh
docker volume create mydata
docker volume ls
docker run -v mydata:/var/lib/data myapp     # named volume mount
docker run -v $(pwd):/workspace myapp        # bind-mount current dir (host path)
docker run -v $(pwd):/workspace:ro myapp     # read-only
docker volume rm mydata
```

On Windows PowerShell use `${PWD}` instead of `$(pwd)`. With WSL2, paths under `/mnt/c/...` are slow — keep code in `~/...` (the WSL filesystem).

### Networks
```sh
docker network ls
docker network create mynet
docker run -d --name api --network mynet myapi
docker run -d --name web --network mynet -p 8080:80 myweb
# Containers on the same network reach each other by container name as DNS:
#   from `web`, http://api:3000 just works.
```

### Cleanup (disk-space recovery)
```sh
docker system df                             # see what's using space
docker container prune                       # remove all stopped containers
docker image prune                           # remove dangling images (untagged)
docker image prune -a                        # ⚠ remove ALL images not used by a container
docker volume prune                          # ⚠ remove unused volumes (data loss possible)
docker system prune                          # combined: containers + dangling images + networks
docker system prune -a --volumes             # ⚠ nuke everything unused (use with care)
```

## Dockerfile basics

```dockerfile
# Pin the base image — never use `:latest` in committed Dockerfiles
FROM node:20-alpine AS build

# Set working dir; persists for subsequent commands
WORKDIR /app

# Copy dependency manifest first → leverages layer cache when only source changes
COPY package*.json ./
RUN npm ci

# Now copy source
COPY . .
RUN npm run build

# ---- Multi-stage: smaller final image ----
FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
COPY package.json ./

ENV NODE_ENV=production
EXPOSE 3000
USER node                                    # don't run as root in production

# JSON-array form (exec form) → no shell, signals reach the process
CMD ["node", "dist/server.js"]
```

### Build optimization — layer cache

Order Dockerfile commands **least-to-most-changing**: pin base → install OS packages → copy and install deps → copy source → build. Each `COPY` / `RUN` is a layer, cached per command. Changing `src/index.js` shouldn't invalidate `npm ci`.

`.dockerignore` (sibling to Dockerfile) keeps `node_modules`, `.git`, build artifacts, secrets out of the build context:
```
node_modules
.git
.env
dist
*.log
.vscode
```

## Docker Compose

`docker-compose.yml` (or `compose.yaml`) declares a multi-container stack:

```yaml
services:
  web:
    build: .
    ports:
      - "8080:80"
    environment:
      - DATABASE_URL=postgres://postgres:secret@db:5432/app
    depends_on:
      db:
        condition: service_healthy
    volumes:
      - ./src:/app/src

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: app
    volumes:
      - dbdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD", "pg_isready", "-U", "postgres"]
      interval: 5s
      timeout: 3s
      retries: 5

volumes:
  dbdata:
```

```sh
docker compose up                            # foreground
docker compose up -d                         # detached
docker compose up --build                    # rebuild before starting
docker compose ps                            # status
docker compose logs -f web                   # follow one service's logs
docker compose exec web bash                 # shell into a running service
docker compose run --rm web npm test         # one-off: spin up, run, remove
docker compose down                          # stop + remove containers + network
docker compose down -v                       # ⚠ also remove named volumes (data loss)
```

## Common workflows

**Quick interactive test of a Linux command without installing it:**
```sh
docker run --rm -it -v $(pwd):/work -w /work alpine sh
```

**One-shot Python REPL with deps:**
```sh
docker run --rm -it python:3.12-slim bash -c "pip install requests && python"
```

**Build, tag, push to GHCR:**
```sh
echo $TOKEN | docker login ghcr.io -u <user> --password-stdin
docker build -t ghcr.io/<user>/myapp:1.2.0 .
docker push ghcr.io/<user>/myapp:1.2.0
```

**Debug a failing container:**
```sh
docker logs --tail 200 <name>                # what did it print before dying?
docker inspect <name> --format '{{.State.ExitCode}} {{.State.Error}}'
docker run --rm -it --entrypoint sh <image>  # poke around inside the image
```

## Gotchas

- **`:latest` is not a version pin** — it just means "whatever the registry currently calls latest". Always pin a real tag (`node:20.11-alpine3.19`) in production.
- **Build context** is the entire directory you pass to `docker build`. Without `.dockerignore`, you ship `node_modules` and `.git` into every layer's cache. Slow + leaky.
- **Bind mounts on Windows / macOS** go through a VM file-sharing layer — significantly slower than Linux. Volumes (named) are faster than bind mounts on these OSes.
- **Container running as root** can write files on bind-mounted host dirs as root, leaving you `chown`ing them later. Add `USER node` (or any non-root) in the Dockerfile, or pass `--user $(id -u):$(id -g)`.
- **Signals**: `CMD ["sh", "-c", "node app.js"]` (shell form) puts a shell as PID 1 — signals don't reach Node. Use exec form `CMD ["node", "app.js"]` so Node is PID 1 and graceful shutdown works.
- **`docker compose` (space) vs `docker-compose` (hyphen)**: the hyphen version is the legacy Python CLI (Compose v1), now deprecated. New work: `docker compose`. Same `docker-compose.yml` file.
- **`docker system prune -a` is destructive**: removes any image not currently used by a container. Don't run it casually on a build machine you care about.
- **Layer cache invalidates on `COPY` source changes** — even file mtimes can matter on some setups. If a build "always rebuilds from scratch", check `.dockerignore` and the order of `COPY` instructions.
- **Resource limits**: by default a container can use all host CPU/memory. Use `--cpus`, `--memory` flags or the `deploy.resources` Compose section in production.

## Where to learn more

- `docker --help`, `docker <subcommand> --help` — built-in help. Comprehensive.
- `docker compose --help`, `docker compose <subcommand> --help`.
- [docs.docker.com](https://docs.docker.com/) — official docs. Especially:
  - [docs.docker.com/get-started](https://docs.docker.com/get-started/) — full beginner walkthrough.
  - [docs.docker.com/build/building/best-practices](https://docs.docker.com/build/building/best-practices/) — Dockerfile best practices.
  - [docs.docker.com/compose](https://docs.docker.com/compose/) — Compose reference.
- [hub.docker.com](https://hub.docker.com/) — official images. Each one's README explains supported tags and config env vars.
- [github.com/docker-library/official-images](https://github.com/docker-library/official-images) — source of truth for what makes an image "official".
- [containertoolbx.org](https://containertoolbx.org/) and [podman.io](https://podman.io/) — drop-in alternatives if Docker Desktop's licensing doesn't fit.
