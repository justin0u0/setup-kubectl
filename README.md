# setup-kubectl

Github actions workflow to setup `kubectl` with Kubernetes service account.

## Usage

See [action.yml](./action.yml) for detail.

**With `kubectl` stable version:**

```
- name: setup kubectl
  uses: justin0u0/setup-kubectl@v1
  with:
    kubectl-version: stable
    cluster-certificate-authority-data: ${{ secrets.KUBERNETES_CLUSTER_CLIENT_CERTIFICATE_AUTHORITY_DATA }}
    cluster-server: ${{ secrets.KUBERNETES_CLUSTER_SERVER }}
    credentials-token: ${{ secrets.KUBERNETES_CREDENTIALS_TOKEN }}
```

**With `kubectl` specific version:**

```
- name: setup kubectl
  uses: justin0u0/setup-kubectl@v1
  with:
    kubectl-version: v1.23.6
    cluster-certificate-authority-data: ${{ secrets.KUBERNETES_CLUSTER_CLIENT_CERTIFICATE_AUTHORITY_DATA }}
    cluster-server: ${{ secrets.KUBERNETES_CLUSTER_SERVER }}
    credentials-token: ${{ secrets.KUBERNETES_CREDENTIALS_TOKEN }}
```

## Setup Service Account in Kubernetes


# Setup kubectl Github Actions

1. Create `ServiceAccount`.

    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: github-actions
    EOF
    ```

2. Create `Role` and `RoleBinding`.
    - If the service account need to have cluster-wide access, use `ClusterRole` and `ClusterRoleBinding`.
    - Add necessary rules for the service account. For example, I only need the `kubectl set image` ,`kubectl rollout status`, `kubectl create job` and `kubectl wait --for=condition=complete job/{job}` commands, so the following rules are sufficient.

    ```bash
    cat <<EOF | kubectl apply -f -
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      name: github-actions
    rules:
    - apiGroups:
      - apps
      - batch
      resources:
      - "*"
      verbs:
      - "get"
      - "list"
      - "watch"
      - "create"
      - "update"
      - "patch"
    ---
    kind: RoleBinding
    apiVersion: rbac.authorization.k8s.io/v1
    metadata:
      name: github-actions
    subjects:
    - kind: ServiceAccount
      name: github-actions
    roleRef:
      kind: Role
      name: github-actions
      apiGroup: rbac.authorization.k8s.io
    EOF
    ```

3. Get the API server CA and the secret token of the service account.

    ```bash
    export SERVICE_ACCOUNT_NAME=github-actions
    export NAMESPACE=default

    # fetch the API server CA
    kubectl get secret/$(kubectl get serviceaccount/github-actions -o json | jq -r '.secrets[0].name') -o json | jq -r '.data["ca.crt"]'

    # fetch the secret token
    kubectl get secret/$(kubectl get serviceaccount/github-actions -o json | jq -r '.secrets[0].name') -o json | jq -r '.data["token"]' | base64 -d
    ```

    Go to the Github repository secrets setting pages: Settings > Secrets > Actions > New Repository secret (or use environment secret).

    - Set `KUBERNETES_CLUSTER_SERVER` to the Kubernetes API server URL.
    - Set `KUBERNETES_CLUSTER_CLIENT_CERTIFICATE_AUTHORITY_DATA` with the API server CA. **Note that the data should be base64 encoded.**
    - Set `KUBERNETES_CREDENTIALS_TOKEN` with the secret token. **Note that the token should NOT be base64 encoded.**

    DO NOT add any newline character when setting the secrets.
