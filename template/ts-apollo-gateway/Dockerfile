FROM node:16.16.0-alpine

RUN addgroup -S app && adduser -S -g app app

# Alternatively use ADD https:// (which will not be cached by Docker builder)
RUN apk --no-cache add curl ca-certificates \
    && echo "Pulling watchdog binary from Github." \
    && curl -sSLf https://github.com/openfaas-incubator/of-watchdog/releases/download/0.4.6/of-watchdog > /usr/bin/fwatchdog \
    && chmod +x /usr/bin/fwatchdog \
    && apk del curl

WORKDIR /root/

# Turn down the verbosity to default level.
ENV NPM_CONFIG_LOGLEVEL warn

RUN mkdir -p /home/app

# Wrapper/boot-strapper
WORKDIR /home/app
COPY --chown=app:app package.json ./ 
COPY --chown=app:app tsconfig.json ./


# This ordering means the npm installation is cached for the outer function handler.
RUN npm i 


# Copy outer function handler
COPY --chown=app:app index.ts ./


# COPY function node packages and install, adding this as a separate
# entry allows caching of npm install

WORKDIR /home/app/function
COPY --chown=app:app function/*.json function/.npmrc* ./

RUN npm i || :

# COPY function files and folders
COPY --chown=app:app function/ ./
RUN chmod 777 /home/app/function

# Set correct permissions to use non root user
WORKDIR /home/app/

RUN npm run build-ts

# chmod for tmp is for a buildkit issue (@alexellis)
RUN chown app:app -R /home/app \
    && chmod 777 /tmp

USER app


ENV cgi_headers="true"
ENV fprocess="node index.js"
ENV mode="http"
ENV upstream_url="http://127.0.0.1:3000"
ENV NODE_ENV production


ENV exec_timeout="10s"
ENV write_timeout="15s"
ENV read_timeout="15s"

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]

