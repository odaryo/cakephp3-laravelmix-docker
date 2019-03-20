ARG NODE_VERSION

FROM node:${NODE_VERSION}-alpine

RUN apk update \
    && apk add --no-cache \
        bash \
    && npm install -g npm yarn \
    && npm install --global gulp-cli
    #&& npm install --global vue-cli

# Setup working directory
ARG APP_CONTAINER
WORKDIR ${APP_CONTAINER}