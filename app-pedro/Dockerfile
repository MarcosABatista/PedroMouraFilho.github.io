# syntax=docker/dockerfile:1

# =========================================================================
# Dockerfile — Site Nuxt (app-pedro) servido como estático via Nginx
# -------------------------------------------------------------------------
# O projeto é gerado como site estático (pnpm generate -> .output/public)
# e servido por um Nginx enxuto. Multi-stage para a imagem final ser leve.
# =========================================================================

# ---------- Estágio 1: build do site estático ----------
FROM node:24-alpine AS builder

# pnpm via corepack (versão fixada no package.json -> packageManager)
RUN corepack enable

WORKDIR /app

# Instala dependências primeiro para aproveitar o cache de camadas
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml ./
RUN pnpm install --frozen-lockfile

# Copia o restante do código e gera o site estático
COPY . .
RUN pnpm generate

# ---------- Estágio 2: imagem final com Nginx ----------
FROM nginx:1.27-alpine AS runner

# Config do Nginx escutando na porta 82
COPY nginx.conf /etc/nginx/conf.d/default.conf

# Artefato estático gerado no estágio anterior
COPY --from=builder /app/.output/public /usr/share/nginx/html

EXPOSE 82

# Healthcheck simples: verifica se o Nginx responde na porta 82
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- http://127.0.0.1:82/ >/dev/null 2>&1 || exit 1

CMD ["nginx", "-g", "daemon off;"]
