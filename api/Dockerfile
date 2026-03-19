FROM node:lts-alpine

RUN apk add --no-cache curl

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY app.js ./

USER node

EXPOSE 3000

CMD ["node", "app.js"]
