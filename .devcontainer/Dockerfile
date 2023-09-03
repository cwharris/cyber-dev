ARG BASE

FROM ${BASE} as base

ARG RAPIDS
ENV RAPIDS=${RAPIDS}

ARG CUDA
ENV CUDA_VERSION="${CUDA_VERSION:-${CUDA}}"

ENV DEFAULT_VIRTUAL_ENV="morpheus"
ENV PYTHON_PACKAGE_MANAGER="conda"

ARG SCCACHE_REGION="us-east-2"
ENV SCCACHE_REGION="$SCCACHE_REGION"
ARG SCCACHE_BUCKET="rapids-sccache-devs"
ENV SCCACHE_BUCKET="$SCCACHE_BUCKET"
ARG VAULT_HOST="https://vault.ops.k8s.rapids.ai"
ENV VAULT_HOST="$VAULT_HOST"

ENV HISTFILE="/home/coder/.cache/._bash_history"

RUN mkdir -p /home/coder/.{aws,cache,config/pip,local,out}

# ===== install common packages ==================================================================

RUN <<EOF

set -x

apt-get update;

# apt-get install -y --no-install-recommends  \
#     libboost-{regex,system,filesystem}-dev  \
#     ;

rm -rf /tmp/*;
rm -rf /var/tmp/*;
rm -rf /var/cache/apt/*;
rm -rf /var/lib/apt/lists/*;

EOF

# ===== install node + camouflage ================================================================

COPY --from=node /usr/local/bin/node /usr/local/bin/node
COPY --from=node /usr/local/include/node /usr/local/include/node
COPY --from=node /usr/local/lib/node_modules /usr/local/lib/node_modules
COPY --from=node /opt/yarn-v*/bin/* /usr/local/bin/
COPY --from=node /opt/yarn-v*/lib/* /usr/local/lib/

RUN \
<<EOF

bash -c 'echo -e "
fund=false\n
audit=false\n
save-prefix=\n
--omit=optional\n
save-exact=true\n
package-lock=false\n
update-notifier=false\n
scripts-prepend-node-path=true\n
registry=https://registry.npmjs.org/\n
" | tee /root/.npmrc >/dev/null'

ln -s /usr/local/bin/node /usr/local/bin/nodejs
ln -s /usr/local/lib/node_modules/npm/bin/npm-cli.js /usr/local/bin/npm
ln -s /usr/local/lib/node_modules/npm/bin/npx-cli.js /usr/local/bin/npx

echo "node version: $(node --version)"
echo " npm version: $(npm --version)"
echo "yarn version: $(yarn --version)"

npm install -g camouflage-server@0.9

EOF

FROM base as devel
FROM base as runtime