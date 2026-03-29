This repository is a minimal Node.js (Express) web app packaged for Docker and deployed via Jenkins/Render.

Key files
- `server.js` — single-file Express app. Listens on PORT (env) or 3000 by default and responds to GET / with a simple message.
- `package.json` — contains a single script: `start: node server.js` and dependency `express@4.21.1`.
- `Dockerfile` — builds the image using `node:24.11.1`, runs `npm install`, exposes port 3000 and starts `node server.js`.
- `Jenkinsfile` — CI pipeline: logs into Docker Hub and pushes image `mosazhaw/node-web-app`, then triggers a Render deployment via a curl call.

Big picture / architecture
- Very small monolith: a single Node process (Express) that serves HTTP. All app code lives in `server.js`.
- Packaging: app is containerized via the `Dockerfile` and pushed to Docker Hub as `mosazhaw/node-web-app` from Jenkins.
- Deployment: Jenkins pipeline expects credentials and uses the Render API to trigger an external deployment.

Developer workflows (how to run, build, debug)
- Run locally (node):
  - `npm start` (uses `node server.js`). The app reads `process.env.PORT` or defaults to 3000.
- Run in Docker (build + run):
  - Build: `docker build -t mosazhaw/node-web-app .` (also shown in the Dockerfile comments).
  - Run: `docker run -p 3000:3000 --name expressapp -d mosazhaw/node-web-app`
- Jenkins CI expectations (important for local simulation):
  - The pipeline logs into Docker Hub using credentials id `DockerHub-mosazhaw` and pushes the `mosazhaw/node-web-app` image.
  - The pipeline sets `DOCKER_HOST=tcp://host.docker.internal:2375` before using Docker; CI assumes access to a remote Docker daemon at that socket.
  - After pushing, the pipeline triggers Render with credential id `RenderDeployKey` by calling `curl https://api.render.com/deploy/$KEY`.

Project-specific patterns / conventions
- Minimal single-file app: extend `server.js` for simple features or add a `src/` directory and update `package.json` main/script if splitting.
- Hard-coded image name: Jenkinsfile pushes `mosazhaw/node-web-app`. If you change the image name, update `Jenkinsfile` and any deployment hooks.
- Credentials referenced in Jenkinsfile are stored in Jenkins and referenced by id — do not hardcode secrets into files.

Integration points & external dependencies
- Docker Hub: image `mosazhaw/node-web-app` (pushed by Jenkins).
- Render: deployment trigger via the Render API using the `RenderDeployKey` in Jenkins.
- Docker daemon access: Jenkinsfile expects a reachable Docker socket at `host.docker.internal:2375` (CI-specific detail).

Examples to copy/paste
- Start locally with port override:
  - `PORT=4000 npm start` → listens on 4000
- Build and run container with different host port:
  - `docker build -t mosazhaw/node-web-app .`
  - `docker run -p 3001:3000 mosazhaw/node-web-app`

What an AI agent should not change without explicit instruction
- Do not replace `Jenkinsfile` credential ids or the Docker image name without confirmation — CI and deployment rely on these exact values.
- Avoid adding secrets or API keys directly into repo files. Jenkins uses credential storage.

Suggested small, safe improvements (ask before applying)
- Add a `Makefile` or extra NPM scripts for build/test/run to make local dev and CI simulation easier.
- Add a `README.md` with the same commands and environment notes so humans and bots have the same reference.

If anything in this file is unclear or you want examples expanded (tests, alternative CI, or a multi-service layout), tell me which section to expand.
