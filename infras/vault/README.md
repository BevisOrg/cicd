# HashiCorp Vault (Helm)

```sh
helm repo add hashicorp https://helm.releases.hashicorp.com

helm repo update

helm upgrade --install vault hashicorp/vault \
-n vault \
-f infras/vault/values.yaml \
--create-namespace
```

## Document

- [Vault trên Kubernetes](https://developer.hashicorp.com/vault/docs/platform/k8s)
- [vault-helm GitHub](https://github.com/hashicorp/vault-helm)
