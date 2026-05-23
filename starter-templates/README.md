# starter-templates

**Strategy: use battle-tested scaffolders + apply our rules — don't maintain frozen boilerplates.**

The original Week 2 plan called for committing full starter projects to this directory (MERN, Flutter, SwiftUI, Go, auth, API, Docker). After thinking it through:

- ✋ **Frozen boilerplates go stale fast.** A React starter committed today will have outdated deps in 3 months, deprecated patterns in 6.
- ✋ **Maintaining 7 starters is a job, not a side project.** Every dep update needs to be propagated across all of them.
- ✅ **Community scaffolders are already excellent and stay current.** Use them, then apply our rules on top.

## How to start a new project (per stack)

### MERN

```bash
# Backend
npm create vite@latest my-api -- --template node-ts   # or use Hono / express-generator
cd my-api && npm i express mongoose zod dotenv pino

# Frontend
npm create vite@latest my-app -- --template react-ts
cd my-app && npm i @tanstack/react-query react-hook-form @hookform/resolvers zod tailwindcss
```

Then drop into the project root:
- Copy `ai-memory/rules/node-rules.md` (backend) or `react-rules.md` (frontend)
- Create `.aider.conf.yml` with that rule in `read:`

### Flutter

```bash
flutter create my_app --org com.lokeshburade --platforms ios,android
cd my_app
flutter pub add hooks_riverpod riverpod_annotation freezed_annotation dio go_router
flutter pub add --dev build_runner riverpod_generator freezed json_serializable
```

Apply rules from `ai-memory/rules/flutter-rules.md`.

### SwiftUI

In Xcode: `File → New → Project → iOS → App`. Choose SwiftUI + Swift + no Core Data (use SwiftData if needed).

After scaffold, add Swift Package Manager dependencies as needed: `swift-snapshot-testing` for snapshot tests, `KeychainAccess` for token storage.

Apply rules from `ai-memory/rules/swiftui-rules.md`.

### Go backend

```bash
mkdir my-service && cd my-service
go mod init github.com/Lokeshburade007/my-service
go get -u github.com/go-chi/chi/v5 github.com/jackc/pgx/v5 github.com/joho/godotenv
mkdir -p cmd/api internal/{domain,usecase,infrastructure,delivery/http,config}
```

Apply rules from `ai-memory/rules/golang-rules.md`.

### Auth boilerplate (Node + Express + Mongo)

Not a separate project — drop these files into a MERN backend:

- JWT access + refresh tokens (15min / 7day)
- bcrypt password hashing (cost factor 12)
- Refresh-token rotation, stored hashed in DB
- Rate-limit middleware on `/login` + `/signup`

Generate with: `aider --read ai-memory/rules/node-rules.md --read ai-prompts/prompts/specialized/security-review.md --message "scaffold auth: signup, login, refresh, logout endpoints with JWT + bcrypt + refresh token rotation"`

### Docker template

For any Node service:

```dockerfile
# Dockerfile
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine
WORKDIR /app
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/package*.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/index.js"]
```

```yaml
# compose.yaml (dev)
services:
  api:
    build: .
    ports: ["3000:3000"]
    environment:
      MONGO_URI: mongodb://mongo:27017/app
    depends_on: [mongo]
  mongo:
    image: mongo:7
    volumes: ["mongo-data:/data/db"]
volumes:
  mongo-data:
```

For Go: same pattern with `golang:1.22-alpine` build stage and `gcr.io/distroless/static-debian12` runtime.

## When you DO commit a starter here

Reserve this folder for boilerplates that:
- Encode a **non-obvious** pattern you've battle-tested and want to reuse exactly.
- Don't have an equivalent community scaffolder.
- You'll maintain at least once a quarter.

Otherwise, don't add bit-rot.
