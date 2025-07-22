# ---------- Stage 1: build your fork ----------
FROM node:20-alpine AS build
WORKDIR /app

# speed: turn on pnpm
RUN corepack enable && corepack prepare pnpm@latest --activate

# copy repo (context MUST be the root of your fork)
COPY . .

# install & build
ENV NODE_OPTIONS="--max-old-space-size=4096"
RUN pnpm install --frozen-lockfile && pnpm build

# ---------- Stage 2: runtime ----------
FROM node:20-alpine
WORKDIR /app

# runtime deps (pdf/image tools etc.)
RUN apk add --no-cache chromium git python3 py3-pip make g++ build-base cairo-dev pango-dev curl
ENV PUPPETEER_EXECUTABLE_PATH=/usr/bin/chromium-browser

# copy compiled artifacts
COPY --from=build /app/packages/server/dist        ./packages/server/dist
COPY --from=build /app/packages/ui/dist            ./packages/ui/dist
COPY --from=build /app/packages/components/dist    ./packages/components/dist
COPY --from=build /app/node_modules                ./node_modules
COPY --from=build /app/package.json                ./package.json

# OPTIONAL: hard-remove the 2 crashy nodes (if you didn't pin langchain)
RUN rm -rf ./packages/components/dist/nodes/chatmodels/ChatFireworks \
           ./packages/components/dist/nodes/chatmodels/ChatTogetherAI

ENV PORT=3000
EXPOSE 3000

CMD ["node", "packages/server/dist/index.js"]
