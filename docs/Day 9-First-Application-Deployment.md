# Day 9 — First Application Deployment

## Phase 2 — Docker Infrastructure
### Week 3 — Containerization

## Objective

Deploy an existing GitHub project as a Docker container on the Raspberry Pi and verify it from the Mac over the local network.

## Application

Repository: `https://github.com/BhavyeshAsapu/Domain_Home_page.git`

Application directory:

```text
/srv/apps/Domain_Home_page
```

The project is a React/TypeScript application using Vite and TanStack Start. The repository uses Bun and contains a `bun.lock` lockfile.

## Day 9 Checklist

- [x] Choose one existing GitHub project
- [x] Create Dockerfile
- [x] Build Docker image
- [x] Run application locally
- [x] Run application on Pi
- [x] Expose application internally
- [x] Verify application from Mac
- [x] Document deployment process

---

## 1. Choose Existing GitHub Project

The `Domain_Home_page` repository was selected as the first real Docker deployment because it is a relatively simple application.

It was cloned to:

```text
/srv/apps/Domain_Home_page
```

Git status confirmed the repository was on the `main` branch and synchronized with the remote repository.

---

## 2. Initial Environment Check

The Raspberry Pi did not have Node.js or npm installed directly on the host.

```bash
node --version
npm --version
```

Both returned:

```text
-bash: node: command not found
-bash: npm: command not found
```

This was acceptable because Docker would provide the application build environment.

The project was inspected:

```bash
ls -la package.json package-lock.json bun.lock
```

The repository contained:

```text
package.json
bun.lock
```

There was no `package-lock.json`.

Therefore, the project uses Bun for dependency management.

---

## 3. Initial Dockerfile Attempt

The first Dockerfile assumed a normal Vite production output:

```text
/app/dist
```

The Docker build failed with:

```text
"/app/dist": not found
```

The application did not generate a `dist` directory.

---

## 4. Investigating the Build Output

The application was inspected using a temporary Docker debugging stage.

The build produced:

```text
/app/.output/
├── server/
└── public/
```

The server output contained:

```text
/app/.output/server/index.mjs
```

The public assets were under:

```text
/app/.output/public
```

This confirmed that the application is a TanStack Start application with a server-side production output rather than a simple static Vite site.

Therefore, Nginx alone was not used for the final runtime.

---

## 5. Package Manager Correction

The first production Dockerfile used:

```dockerfile
RUN npm ci
```

The build failed because the repository did not contain a `package-lock.json`.

The error was:

```text
The `npm ci` command can only install with an existing package-lock.json
```

The repository contained:

```text
bun.lock
```

Therefore, the Dockerfile was changed to use Bun.

---

## 6. Final Dockerfile

```dockerfile
# Stage 1: Build the application
FROM oven/bun:1 AS builder

WORKDIR /app

COPY package.json bun.lock ./

RUN bun install --frozen-lockfile

COPY . .

RUN bun run build


# Stage 2: Production server
FROM oven/bun:1-slim AS runner

WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000
ENV HOST=0.0.0.0

COPY --from=builder /app/.output ./.output

EXPOSE 3000

CMD ["bun", ".output/server/index.mjs"]
```

### Why a multi-stage build was used

The builder stage contains the tools required to install dependencies and build the application.

The runner stage contains the runtime environment and generated production output.

```text
GitHub repository
       |
       v
Builder container
       |
       | bun install
       | bun run build
       v
/app/.output
       |
       v
Production container
       |
       v
Bun + TanStack Start server
```

---

## 7. Docker Ignore File

A `.dockerignore` file was created:

```text
node_modules
dist
.git
.gitignore
README.md
```

This prevents unnecessary files from being sent to the Docker build context.

---

## 8. Build Docker Image

The final image was built with:

```bash
sudo docker build --no-cache -t domain-home-page:v1 .
```

The build completed successfully.

Important stages included:

```text
[builder] COPY package.json bun.lock ./
[builder] RUN bun install --frozen-lockfile
[builder] COPY . .
[builder] RUN bun run build
[runner] COPY --from=builder /app/.output ./.output
```

The resulting image was:

```text
domain-home-page:v1
```

Reported image size:

```text
266 MB
```

A temporary debugging image was also created:

```text
domain-home-page:debug
```

The debug image was not required for the final deployment.

---

## 9. Run Application on Raspberry Pi

The application was started with:

```bash
sudo docker run -d   --name domain-home-page   -p 3000:3000   domain-home-page:v1
```

Docker created the container successfully.

The running container was verified:

```bash
sudo docker ps
```

The container showed:

```text
domain-home-page:v1
0.0.0.0:3000->3000/tcp
[::]:3000->3000/tcp
```

Status:

```text
Up
```

---

## 10. Verify Container Logs

```bash
sudo docker logs domain-home-page
```

Output:

```text
Started server: http://localhost:3000
```

This confirmed that the application server started successfully.

---

## 11. Verify Application on Raspberry Pi

The application was tested locally:

```bash
curl -I http://localhost:3000
```

Response:

```text
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
content-length: 0
```

This confirmed that the container was serving HTTP successfully.

---

## 12. Expose Application Internally

The Docker port mapping:

```text
3000:3000
```

made the application available through the Raspberry Pi's network interface.

The Raspberry Pi's LAN address during the final test was:

```text
10.46.3.71
```

The application was therefore accessible internally at:

```text
http://10.46.3.71:3000
```

This was an internal LAN deployment and was not exposed through a public domain.

---

## 13. Verify Application from Mac

From the Mac:

```bash
curl -I http://10.46.3.71:3000
```

Response:

```text
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
content-length: 0
```

This confirmed the complete network path:

```text
Mac
 |
 | HTTP :3000
 v
Raspberry Pi
 |
 | Docker port 3000
 v
domain-home-page container
 |
 v
TanStack Start server
```

The application could also be opened in a browser at:

```text
http://10.46.3.71:3000
```

Note: Markdown syntax such as `[http://10.46.3.71:3000](...)` should not be typed directly into the shell. It is a Markdown link, not a terminal command.

---

## 14. Final Docker State

Running container:

```text
CONTAINER ID   IMAGE                  STATUS       PORTS
cb437191d896   domain-home-page:v1   Up           0.0.0.0:3000->3000/tcp
```

Relevant images:

```text
domain-home-page:v1
domain-home-page:debug
```

The `v1` image was the actual deployment image.

---

## 15. Troubleshooting Lessons

### Problem 1 — `/app/dist` not found

The initial Dockerfile expected:

```text
/app/dist
```

The application instead generated:

```text
/app/.output
```

**Lesson:** Do not assume every Vite-based project produces the same deployment structure. Inspect the framework's actual build output.

### Problem 2 — `npm ci` failed

The repository did not contain:

```text
package-lock.json
```

It contained:

```text
bun.lock
```

**Lesson:** Use the package manager and lockfile that the project actually uses.

The final Docker build therefore uses:

```bash
bun install --frozen-lockfile
```

### Problem 3 — Nginx was not appropriate for the final runtime

The application generated a server runtime:

```text
.output/server/index.mjs
```

Therefore, the final container runs the TanStack Start server using Bun rather than serving only static files through Nginx.

---

## 16. Final Deployment Architecture

```text
                    GitHub
                      |
                      v
             Domain_Home_page
                      |
                      v
                Dockerfile
                      |
             +--------+--------+
             |                 |
             v                 v
       Builder image     Production image
        oven/bun:1       oven/bun:1-slim
             |                 |
       bun install             |
       bun run build           |
             |                 |
             +--------+--------+
                      |
                      v
                 .output/
                      |
                      v
          domain-home-page:v1
                      |
                      v
           Docker container
          domain-home-page
                      |
                 Port 3000
                      |
                      v
             Raspberry Pi
              10.46.3.71
                      |
                      v
                    Mac
```

---

## 17. Final Result

Day 9 was completed successfully.

The existing GitHub application was:

1. Cloned to the Raspberry Pi.
2. Containerized using a multi-stage Dockerfile.
3. Built successfully using Bun.
4. Started as a Docker container.
5. Exposed internally on port 3000.
6. Verified locally on the Raspberry Pi.
7. Verified remotely from the Mac over the LAN.

Final application endpoint used during testing:

```text
http://10.46.3.71:3000
```

## Day 9 Checklist

- [x] Choose one existing GitHub project
- [x] Create Dockerfile
- [x] Build Docker image
- [x] Run application locally
- [x] Run application on Pi
- [x] Expose application internally
- [x] Verify application from Mac
- [x] Document deployment process

**Phase 2 Docker Infrastructure — Day 9 complete.**
