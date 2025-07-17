FROM repos.gbt.gbtad.com:9444/gbt/backstage:20-bookworm-slim-gbt-20240714

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
 --mount=type=cache,target=/var/lib/apt,sharing=locked \
 apt-get update && \
 apt-get install -y --no-install-recommends python3 g++ build-essential && \
 yarn config set python /usr/bin/python3
RUN mkdir -p /home/node/.cache/node-gyp && chown -R node:node /home/node/.cache/node-gyp
USER root
COPY ca-certificates.crt /usr/local/share/ca-certificates/ca-certificates.crt
COPY my-cert1.pem /usr/local/share/ca-certificates/my-cert1.pem
ENV NODE_EXTRA_CA_CERTS=/usr/local/share/ca-certificates/my-cert1.pem
ENV NODE_ENV production
RUN npm config set registry https://repos.gbt.gbtad.com/repository/npmjs/
RUN npm config set cafile /usr/local/share/ca-certificates/ca-certificates.crt

WORKDIR /app

# ⬇️ Copy source files instead of prebuilt tarballs
COPY --chown=root:root package.json ./
COPY --chown=node:node packages packages
COPY --chown=node:node app-config*.yaml ./
COPY --chown=node:node examples ./examples

# ⬇️ Rebuild everything inside the image
RUN yarn install
RUN yarn tsc
RUN yarn build:backend --config app-config.yaml

# ⬇️ Create and extract tarballs inside the image
RUN tar czf skeleton.tar.gz packages/backend/dist/skeleton/ && \
 tar czf bundle.tar.gz packages/backend/dist/ && \
 tar xzf skeleton.tar.gz && rm skeleton.tar.gz && \
 tar xzf bundle.tar.gz && rm bundle.tar.gz

RUN yarn add isolated-vm@5.0.0 --ignore-workspace-root-check || true

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
