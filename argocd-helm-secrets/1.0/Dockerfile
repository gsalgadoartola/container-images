FROM docker.io/argoproj/argocd:v1.5.7

ENV SOPS_VERSION=v3.5.0 \
    VAULT_VERSION=1.4.1 \
    HELM_SECRETS_VERSION=v3.2.0 \
    HELM_SECRETS_DRIVER=vault \
    HELM_PLUGINS=/home/argocd/.local/share/helm/plugins/

USER root

RUN apt-get update && \
    apt-get install -y curl gpg unzip && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    # sops
    curl -o /usr/local/bin/sops -LO https://github.com/mozilla/sops/releases/download/${SOPS_VERSION}/sops-${SOPS_VERSION}.linux && \
    chmod +x /usr/local/bin/sops && \
    # vault
    curl -LO https://releases.hashicorp.com/vault/${VAULT_VERSION}/vault_${VAULT_VERSION}_linux_amd64.zip && \
    unzip vault_${VAULT_VERSION}_linux_amd64.zip && \
    rm -rf vault_${VAULT_VERSION}_linux_amd64.zip && \
    mv vault /usr/local/bin/ && \
    chmod +x /usr/local/bin/vault && \
    # helm
    mv /usr/local/bin/helm /usr/local/bin/_helm

COPY helm-wrapper /usr/local/bin/helm

USER argocd

# helm-secrets
RUN helm plugin install https://github.com/jkroepke/helm-secrets --version ${HELM_SECRETS_VERSION} && \
    sed -i "s#_VAULT_REGEX=.*#_VAULT_REGEX='\!vault [A-Za-z0-9][A-Za-z0-9/\\\_-]*\\\\\#[A-Za-z0-9][A-Za-z0-9_-]*'#" /home/argocd/.local/share/helm/plugins/helm-secrets/scripts/drivers/vault.sh
