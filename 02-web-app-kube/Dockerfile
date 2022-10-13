FROM node:18-alpine3.15 as fe

WORKDIR /fe

COPY package*.json ./

RUN npm install

COPY src ./src

RUN npm run build

COPY static ./static

FROM nginx:1.23.1-alpine

WORKDIR /usr/share/nginx/html

COPY nginx/nginx.conf /etc/nginx/conf.d/default.conf

RUN rm -rf ./*

EXPOSE 80

COPY --from=fe /fe/static .