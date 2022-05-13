# setup-kubectl

Github actions workflow to setup `kubectl` with Kubernetes service account.

## Usage

See [action.yml](./action.yml) for detail.

```
- name: prepare kubectl
  uses: justin0u0/kubectl-action@v1
  with:
    kubectl-version: stable
    cluster-certificate-authority-data: ${{ secrets.KUBERNETES_CLUSTER_CLIENT_CERTIFICATE_AUTHORITY_DATA }}
    cluster-server: ${{ secrets.KUBERNETES_CLUSTER_SERVER }}
    credentials-token: ${{ secrets.KUBERNETES_CREDENTIALS_TOKEN }}
```
