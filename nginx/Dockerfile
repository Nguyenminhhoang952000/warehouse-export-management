FROM node:20-alpine AS fe-builder

WORKDIR /app

COPY warehouse-management-fe/package*.json ./
RUN npm ci

COPY warehouse-management-fe/ ./
RUN npm run build

FROM nginx:1.27-alpine

COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=fe-builder /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
