# Build Image

## Nginx configuration

```nginx
worker_processes auto;
events { worker_connections 1024; }

http {
    server {
        listen 80;

        location / {
            proxy_pass http://frontend:3000; # Use the Docker service name and port
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /api/ {
            # 這個rewrite 會把 http://<hostname>/api/user 變成 http://backend/user
            rewrite ^/api(.*)$ $1 break;
            proxy_pass http://backend:3000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```


## NextJS

#### Note
在這個前端案例當中 Packages裡面的檔案只是devDependience
```Dockerfile
FROM node:23-bookworm-slim AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable

FROM base as builder
WORKDIR /app

#複製會需要用到的Package的Json
COPY lerna.json package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/next-frontend/package.json ./apps/next-frontend/
COPY packages/financemanager-website-types/package.json packages/financemanager-website-types/package-lock.json ./packages/financemanager-website-types/
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

COPY ./apps/next-frontend ./apps/next-frontend
COPY ./packages ./packages
RUN pnpm build

# 只重新下載Production Dependenice
FROM base as prod-deps
WORKDIR /app
COPY lerna.json package.json pnpm-lock.yaml pnpm-workspace.yaml ./
COPY apps/next-frontend/package.json ./apps/next-frontend/
RUN pnpm install --prod --frozen-lockfile

FROM base AS frontend

WORKDIR /app

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

# 因為是PNPM Workspace的關係，要把根目錄的跟appnode_modules都複製進去
COPY --from=prod-deps --chown=nextjs:nodejs /app/node_modules ./node_modules
COPY --from=prod-deps --chown=nextjs:nodejs /app/apps/next-frontend/node_modules ./apps/frontend/node_modules

# 這三行主要是因為使用NextJS
COPY --from=builder --chown=nextjs:nodejs /app/apps/next-frontend/package.json ./apps/frontend/package.json
COPY --from=builder --chown=nextjs:nodejs /app/apps/next-frontend/public ./apps/frontend/public
COPY --from=builder --chown=nextjs:nodejs /app/apps/next-frontend/.next ./apps/frontend/.next

#如果是使用Nuxt的話只需要把build出來的.output檔案複製就好
# COPY --from=build /app/apps/frontend/.output /app/apps/frontend/.output
# CMD [ "node", ".output/server/index.mjs" ]

# 這邊是幫使用者設定一些PNPM相關的東西 如果是Root或是直接使用Node執行的話就不用
ENV COREPACK_HOME=/app/.cache/corepack
RUN mkdir -p $COREPACK_HOME && chown -R nextjs:nodejs $COREPACK_HOME

USER nextjs

EXPOSE 3000

WORKDIR /app/apps/frontend

CMD ["pnpm", "start"]
```

## NestJS

```Dockerfile
FROM node:20-slim AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN corepack enable
COPY . /app
WORKDIR /app

FROM base AS build
RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile
RUN pnpm build

FROM base AS prod-deps
RUN --mount=type=cache,id=pnpm,target=/pnpm/store \ 
    pnpm pkg delete scripts.prepare && \ 
    cd apps/frontend && pnpm pkg delete scripts.postinstall && \
    cd ../.. && pnpm install --prod --frozen-lockfile

FROM node:20-slim AS backend

COPY --from=base /app/packages/classroom-website-types/package.json /app/packages/classroom-website-types/package.json

COPY --from=prod-deps /app/apps/backend/node_modules /app/apps/backend/node_modules
COPY --from=build /app/packages/classroom-website-types/dist /app/packages/classroom-website-types/dist

COPY --from=prod-deps /app/node_modules /app/node_modules
COPY --from=build /app/apps/backend/dist /app/apps/backend/dist

WORKDIR /app/apps/backend
USER node
ENV NODE_ENV production
EXPOSE 3000

CMD [ "node", "dist/src/main.js" ]
```