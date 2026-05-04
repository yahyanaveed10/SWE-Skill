# Container Signals

Dockerfile and container runtime heuristics. Signals for diagnosing problems; not a Dockerfile tutorial.

---

## Image size signals

**Large images (1GB+) without justification:** Most application images should be well under 500MB, often under 100MB with distroless or alpine bases. Large images slow pulls, increase attack surface, and often indicate unnecessary build tools left in the runtime image.

**Ask:** Is this a build artifact being shipped in a runtime image? Use multi-stage builds to separate the build environment from the runtime image.

**Layers that rarely change should come before layers that change frequently.** Docker layer caching is invalidated from the layer that changed downward. A `COPY . .` before `RUN npm install` means every source change invalidates the dependency layer. The correct order: copy dependency manifests → install dependencies → copy source.

---

## Multi-stage build pattern

```
# Stage 1: build
FROM node:20 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: runtime
FROM node:20-alpine AS runtime
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

**What it solves:** Build tools, test dependencies, source files, and intermediate artifacts stay in the builder stage and are not shipped to production. The runtime image contains only what the application needs to run.

---

## Security signals in Dockerfiles

**Running as root (no USER instruction):** Most application containers do not need root. A compromised container running as root has more blast radius than one running as a non-privileged user. Add a USER instruction before CMD.

**Secrets in ENV or ARG at build time:** Environment variables and build args are visible in image history (`docker history`). Do not put API keys, passwords, or certificates in the Dockerfile. Use runtime secret injection (Docker secrets, Kubernetes secrets, environment variables set at runtime — not baked in).

**Pinned base image tags vs. floating tags:** `FROM node:20` is a floating tag — it changes when the upstream image updates. `FROM node:20.12.0` is pinned. For reproducibility and auditability, pin the base image (or use a digest). For automatic security updates, use a managed base image service that pins and scans.

**COPY . . copies everything:** Without a `.dockerignore`, `COPY . .` copies `.git`, `node_modules`, `.env` files, and anything else in the build context. Always have a `.dockerignore`. At minimum exclude: `.git`, `node_modules`, `*.env`, test fixtures, local secrets.

---

## Runtime signals

**Container exits immediately:** Check the CMD/ENTRYPOINT. The command must run in the foreground — a process that daemonizes itself will exit immediately from the container's perspective. PID 1 must stay alive.

**Container OOMKilled:** The container exceeded its memory limit. Check memory limit configuration, look for memory leaks, or increase the limit. Do not just increase the limit without understanding the growth pattern.

**Unhealthy HEALTHCHECK:** A missing or misconfigured HEALTHCHECK means the orchestrator (Kubernetes, ECS) cannot distinguish a running container from a broken one. HEALTHCHECK should test the application's actual readiness — not just that the process is running, but that it can serve requests.

**Port not exposed or wrong port:** EXPOSE is documentation, not enforcement. The important part is that the application listens on the port the orchestrator expects. Mismatches cause silent routing failures.

---

## Container orchestration signals

**Pod CrashLoopBackOff (Kubernetes):** The container starts and fails repeatedly. Check logs (`kubectl logs --previous`). Root causes: application crash on startup, missing environment variables, missing secrets, failed readiness probe, or misconfigured health check.

**Pending pods:** Scheduler cannot place the pod. Root causes: insufficient cluster resources, node selector or affinity rules that cannot be satisfied, PVC not bound.

**ImagePullBackOff:** The image cannot be pulled. Root causes: wrong image name or tag, registry authentication missing, registry rate limiting, network policy blocking egress.
