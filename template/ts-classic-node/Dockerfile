FROM --platform=${TARGETPLATFORM:-linux/amd64} ghcr.io/openfaas/classic-watchdog:0.1.5 as watchdog
FROM --platform=${TARGETPLATFORM:-linux/amd64} node:12-alpine as ship

ARG TARGETPLATFORM
ARG BUILDPLATFORM

COPY --from=watchdog /fwatchdog /usr/bin/fwatchdog
RUN chmod +x /usr/bin/fwatchdog

RUN apk --no-cache add curl ca-certificates \
    && addgroup -S app && adduser -S -g app app

WORKDIR /root/

# Turn down the verbosity to default level.
ENV NPM_CONFIG_LOGLEVEL warn

RUN mkdir -p /home/app/function
RUN chmod 777 /home/app/function

# Wrapper/boot-strapper
WORKDIR /home/app
COPY package.json ./

# Install typescript globally
RUN npm i -g typescript

# This ordering means the npm installation is cached for the outer function handler.
RUN npm i

# Copy outer function handler
COPY index.js ./
COPY tsconfig.json ./

# COPY function node packages and install, adding this as a separate
# entry allows caching of npm install

WORKDIR /home/app/function

COPY function/*.json ./
COPY function/.npmrc ./

RUN npm i || :

# COPY function files and folders
COPY function ./

# Transpile typescript
RUN npx tsc --project ../tsconfig.json

# Run any tests that may be available
RUN npm test

# Set correct permissions to use non root user
WORKDIR /home/app/

# chmod for tmp is for a buildkit issue (@alexellis)
RUN chown app:app -R /home/app \
    && chmod 777 /tmp

USER app

ENV cgi_headers="true"
ENV fprocess="node index.js"
EXPOSE 8080

HEALTHCHECK --interval=3s CMD [ -e /tmp/.lock ] || exit 1

CMD ["fwatchdog"]
