# Steps to Create and Approve a CSR in Kubernetes

### 1. Generate Private Key
First, you generated a private key using OpenSSL:

```bash
openssl genrsa -out nabil.key 2048

#2. Generate CSR

openssl req -new -key nabil.key -out nabil.csr -subj "/CN=nabil"

#3. Base64 Encode the CSR

cat nabil.csr | base64 | tr -d "\n"

#4. Create the CSR YAML File

apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: nabil
spec:
  request: <base64-encoded-csr>
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 86400  # 1 day
  usages:
    - client auth

#5. Apply the CSR YAML File
kubectl apply -f csr.yaml

#6. Approve the CSR
kubectl certificate approve nabil

#7. Check the CSR Status
kubectl get csr

#8. Export the Issued Certificate to a File

kubectl get csr nabil -o yaml > issuecert.yaml
```