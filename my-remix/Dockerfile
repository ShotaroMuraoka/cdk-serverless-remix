FROM node:22-bookworm-slim AS base

FROM base AS deps
RUN mkdir /app
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

FROM base AS builder

RUN mkdir /app
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NODE_ENV production
RUN npm run build

FROM base AS runner

# lambda-web-adapter
COPY --from=public.ecr.aws/awsguru/aws-lambda-adapter:0.5.0 /lambda-adapter /opt/extensions/lambda-adapter

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 remix

RUN mkdir /app
WORKDIR /app

RUN chown remix:nodejs ./
USER remix

COPY --from=builder --chown=remix:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=remix:nodejs /app/build ./build
COPY --from=builder --chown=remix:nodejs /app/public ./public
COPY --from=builder --chown=remix:nodejs /app/package.json /app/package-lock.json ./

ENV NODE_ENV production
ENV PORT 3000
EXPOSE 3000

CMD ["npm", "run", "start"]