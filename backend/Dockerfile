# Stage 1
FROM node:21 AS backend-builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm config set fetch-timeout 600000 && \
    npm config set registry https://registry.yarnpkg.com && \
    npm i
COPY . .
RUN npm run test || { echo 'Tests failed'; exit 1; }

# Stage 2
FROM node:21-slim
WORKDIR /app
COPY --from=backend-builder /app/node_modules ./node_modules
COPY --from=backend-builder /app/package.json ./
COPY --from=backend-builder /app/dist ./dist # Adjust based on your app's output
COPY .env.docker .env
EXPOSE 8080
CMD ["npm", "start"]




# # Stage 1
# FROM node:21 AS backend-builder

# # setup the working dir
# WORKDIR /app

# # code
# COPY . .

# # packages install
# RUN npm install -g npm@latest && npm i --force

# # tests
# RUN npm run test

# # Stage 2
# FROM node:21-slim

# # setup the working dir
# WORKDIR /app

# # copy the above stage as compressed
# COPY --from=backend-builder /app .

# COPY .env.docker .env

# # Port
# EXPOSE 8080

# # App
# CMD ["npm", "start"]
