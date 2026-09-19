# ---------- Build stage: compile the Vite/React frontend ----------
FROM node:22-alpine AS build
WORKDIR /app

COPY package*.json ./
# Symfony-style merge: install all deps to build the frontend.
RUN npm install

COPY . .
RUN npm run build

# ---------- Runtime stage: serve API + built frontend ----------
FROM node:22-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production

# better-sqlite3 ships prebuilt binaries; keep build tools only if needed.
RUN apk add --no-cache python3 make g++

COPY package*.json ./
RUN npm install --omit=dev

COPY --from=build /app/dist ./dist
COPY backend ./backend
COPY database ./database

# Non-root user for security
RUN addgroup -S app && adduser -S app -G app
RUN chown -R app:app /app
USER app

EXPOSE 5000
CMD ["node", "backend/server.js"]
