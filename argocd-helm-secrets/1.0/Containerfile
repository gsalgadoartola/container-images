#!/usr/bin/env bash

set -e
set -x

IMAGE_NAME=$1
SOPS_VERSION=v3.5.0
VAULT_VERSION=1.4.1
HELM_SECRETS_VERSION=v3.2.0

containerID=$(buildah from docker.io/argoproj/argocd:v1.5.7)

trap "buildah rm $containerID" EXIT

buildah config --user root "$containerID"

buildah run "$containerID" -- apt-get update
buildah run "$containerID" -- apt-get install -y curl gpg unzip
buildah run "$containerID" -- apt-get clean
buildah run "$containerID" -- rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# sops
buildah run "$containerID" -- curl -o /usr/local/bin/sops -LO https://github.com/mozilla/sops/releases/download/${SOPS_VERSION}/sops-${SOPS_VERSION}.linux
buildah run "$containerID" -- chmod +x /usr/local/bin/sops

# vault
buildah run "$containerID" -- curl -LO https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip
buildah run "$containerID" -- unzip vault_${VAULT_VERSION}_linux_amd64.zip
buildah run "$containerID" -- rm -rf vault_${VAULT_VERSION}_linux_amd64.zip
buildah run "$containerID" -- mv vault /usr/local/bin/
buildah run "$containerID" -- chmod +x /usr/local/bin/vault

# helm
buildah run "$containerID" -- mv /usr/local/bin/helm /usr/local/bin/_helm

buildah copy "$containerID" helm-wrapper /usr/local/bin/helm

buildah config --user argocd "$containerID"
buildah config --env HELM_SECRETS_DRIVER=vault --env HELM_PLUGINS=/home/argocd/.local/share/helm/plugins/ "$containerID"

# helm-secrets
buildah run "$containerID" -- helm plugin install https://github.com/jkroepke/helm-secrets --version ${HELM_SECRETS_VERSION}
buildah run "$containerID" -- sed -i "s#_VAULT_REGEX=.*#_VAULT_REGEX='\!vault [A-Za-z0-9][A-Za-z0-9/\\\_-]*\\\\\#[A-Za-z0-9][A-Za-z0-9_-]*'#" /home/argocd/.local/share/helm/plugins/helm-secrets/scripts/drivers/vault.sh

buildah commit "$containerID" "${IMAGE_NAME}"