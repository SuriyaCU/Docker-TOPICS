# Multi-Stage Docker Builds & Distroless Images

---

## What is a Multi-Stage Docker Build?

### Simple Definition
 **Multi-stage Docker build uses multiple `FROM` statements in a single Dockerfile to separate build-time and runtime dependencies.**
 Goal: **smaller, safer, cleaner images**

---

## Why do we need Multi-Stage Builds?

### Traditional Dockerfile (Problem)

```dockerfile
FROM node:18
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["node", "dist/app.js"]
```

Problems:

* Includes:

  * npm
  * build tools
  * source code
* Image is **large**
* More **security risks**

---

## Multi-Stage Build (Solution)

### How it works

* **Stage 1** → Build
* **Stage 2** → Runtime
* Only required files are copied

---

## Multi-Stage Example (Node.js)

```dockerfile
# -------- Stage 1: Build --------
FROM node:18 AS builder
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . .
RUN npm run build

# -------- Stage 2: Runtime --------
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/app.js"]
```

✔ Build tools removed
✔ Smaller image
✔ Faster startup

---

## Multi-Stage Flow

```
Source Code
   ↓
Builder Image (with compilers)
   ↓
Final Image (runtime only)
```

---

## Key Benefits

| Benefit           | Explanation        |
| ----------------- | ------------------ |
| Smaller image     | Only runtime files |
| Better security   | No compilers       |
| Faster deployment | Less data          |
| Clean images      | No junk files      |

---

## What is a Distroless Image?

> **Distroless images contain only the application and its runtime — no shell, no package manager, no OS utilities.**

---

## What Distroless DOES NOT Have

 bash
 apt / yum
 ls, cat, ps
 shell access

HAS Only runtime libraries

---

## Why Distroless?

| Reason           | Benefit                     |
| ---------------- | --------------------------- |
| Very small       | Faster pull                 |
| More secure      | Huge attack surface removed |
| Production-ready | Immutable                   |

---

## Distroless vs Alpine

| Feature         | Alpine | Distroless |
| --------------- | ------ | ---------- |
| Shell           | Yes    | ❌ No       |
| Package manager | Yes    | ❌ No       |
| Size            | Small  | Smaller    |
| Debugging       | Easy   | Hard       |
| Security        | Good   | Excellent  |

📌 **Alpine → dev/testing**
📌 **Distroless → production**

---

## Multi-Stage + Distroless (Best Practice)

### Java Example (Production-Grade)

```dockerfile
# ---------- Build Stage ----------
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn package -DskipTests

# ---------- Runtime Stage ----------
FROM gcr.io/distroless/java17-debian12
WORKDIR /app
COPY --from=build /app/target/app.jar app.jar
CMD ["app.jar"]
```

✔ No shell
✔ No Maven
✔ Only JVM + app

---

## How do you debug Distroless images?

### ❗ Important Interview Question

❌ You cannot exec into container:

```bash
docker exec -it container bash  # ❌ fails
```

### Debug Options

1. Use **sidecar container**
2. Use **debug image (alpine)**
3. Add temporary shell (only for debugging)

---

## When should you use these?

| Scenario               | Recommendation           |
| ---------------------- | ------------------------ |
| Learning / Dev         | Normal images            |
| CI/CD                  | Multi-stage              |
| Production             | Multi-stage + Distroless |
| Security-critical apps | Distroless               |

---

## Real Production Pipeline

```
Code
 ↓
Multi-Stage Docker Build
 ↓
Distroless Runtime Image
 ↓
Registry
 ↓
Kubernetes
```
