# Build stage
FROM node:20-alpine AS builder
WORKDIR /app

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy application source
COPY . .

# Pass API URL as a build argument so Vite builds with the correct API endpoint
ARG VITE_FLASK_API_URL=http://localhost:5000
ENV VITE_FLASK_API_URL=$VITE_FLASK_API_URL

# Build the Vite application
RUN npm run build

# Serve stage (Using unprivileged Nginx image)
FROM nginxinc/nginx-unprivileged:alpine
COPY --from=builder --chown=nginx:nginx /app/dist /usr/share/nginx/html

# Add nginx config for React Router fallback
USER root
RUN echo 'server { \
    listen 8080; \
    location / { \
        root   /usr/share/nginx/html; \
        index  index.html index.htm; \
        try_files $uri $uri/ /index.html; \
    } \
}' > /etc/nginx/conf.d/default.conf
USER nginx

# Unprivileged nginx listens on 8080
EXPOSE 8080
CMD ["nginx", "-g", "daemon off;"]
