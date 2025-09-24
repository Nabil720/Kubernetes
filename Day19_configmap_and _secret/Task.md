# Kubernetes Secret with envFrom - Demo

This guide demonstrates how to create a **Kubernetes Secret** and use it in a **Pod** via `envFrom`. It shows how to securely pass sensitive information (like username and password) as environment variables into a container.

---

## 1. Create the Secret

Create a file called `secret.yaml` with the following content:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
type: Opaque
data:
  username: bXktYXBw          # base64 encoded "my-app"
  password: Mzk1MjgkdmRnN0pi  # base64 encoded "39528$vdg7Jb"


# Apply
kubectl apply -f secret.yaml
kubectl get secret

# Pod
apiVersion: v1
kind: Pod
metadata:
  name: envfrom-secret
spec:
  containers:
  - name: envars-test-container
    image: nginx
    envFrom:
    - secretRef:
        name: test-secret

# Apply
kubectl apply -f pod-secret-envFrom.yaml

# Enter the Pod
kubectl exec -it envfrom-secret -- sh
echo "username: $username"
echo "password: $password"
```