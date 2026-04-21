FROM node:16.15.1

MAINTAINER Opstree Solutions

COPY . /app

WORKDIR /app/

RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    sed -i 's/security.debian.org/archive.debian.org/g' /etc/apt/sources.list && \
    apt-get update -y && \
    apt-get install git -y

RUN npm install --legacy-peer-deps --fetch-timeout=600000

RUN npm run build

RUN yarn global add serve

EXPOSE 3000

ENTRYPOINT ["serve", "-s", "build"]
