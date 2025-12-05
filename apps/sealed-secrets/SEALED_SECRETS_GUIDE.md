# Sealed Secrets Workflow Guide

This guide shows how to create and manage encrypted secrets using Sealed Secrets in your GitOps workflow.

## 🔐 **Setup Sealed Secrets**

### 1. Deploy Sealed Secrets Controller
The controller is already included in your App-of-Apps pattern:
```bash
# It will be deployed automatically when you apply app-of-apps.yaml
kubectl get pods -n kube-system | grep sealed-secrets
```

### 2. Install kubeseal CLI
```bash
# Linux/macOS
wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.24.0/kubeseal-0.24.0-linux-amd64.tar.gz
tar -xvzf kubeseal-0.24.0-linux-amd64.tar.gz
sudo install -m 755 kubeseal /usr/local/bin/kubeseal

# Windows (using Chocolatey)
choco install kubeseal

# Or download binary from GitHub releases
```

## 🔧 **Creating Encrypted Secrets**

### Step 1: Create Regular Secret (Don't commit this!)
```bash
# Create a temporary secret file
cat > jenkins-dev-secret.yaml <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: jenkins-dev-secrets
  namespace: jenkins-dev
type: Opaque
data:
  admin-password: $(echo -n "dev-admin123" | base64)
EOF
```

### Step 2: Encrypt with kubeseal
```bash
# Encrypt the secret
kubeseal --format yaml < jenkins-dev-secret.yaml > jenkins-dev-sealedsecret.yaml

# Clean up the unencrypted file
rm jenkins-dev-secret.yaml
```

### Step 3: Commit Encrypted Secret
```bash
# The SealedSecret is safe to commit to Git
git add apps/jenkins/overlays/dev/jenkins-dev-sealedsecret.yaml
git commit -m "Add encrypted Jenkins dev password"
git push
```

## 🎯 **Quick Commands for Jenkins Secrets**

### Dev Environment Secret
```bash
# Create dev secret
echo -n "dev-admin123" | kubeseal --raw \
  --from-file=/dev/stdin \
  --name jenkins-dev-secrets \
  --namespace jenkins-dev \
  --controller-name sealed-secrets-controller \
  --controller-namespace kube-system
```

### Production Environment Secret
```bash
# Create prod secret
echo -n "super-secure-prod-password" | kubeseal --raw \
  --from-file=/dev/stdin \
  --name jenkins-prod-secrets \
  --namespace jenkins-prod \
  --controller-name sealed-secrets-controller \
  --controller-namespace kube-system
```

## 📝 **Complete SealedSecret Example**

### Replace ENCRYPTED_VALUE_HERE in your files:

**For Dev Environment:**
```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: jenkins-dev-secrets
  namespace: jenkins-dev
spec:
  encryptedData:
    admin-password: "AgBy3i4OJSWK+PiTySYZZA9rO43cGDEQAx..."  # Your encrypted value
  template:
    metadata:
      name: jenkins-dev-secrets
      namespace: jenkins-dev
```

**For Prod Environment:**
```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: jenkins-prod-secrets
  namespace: jenkins-prod
spec:
  encryptedData:
    admin-password: "AgAR5u8KLJHG+TiXcYWEFq2rK54fBHDFVz..."  # Your encrypted value
  template:
    metadata:
      name: jenkins-prod-secrets
      namespace: jenkins-prod
```

## 🔄 **Workflow Integration**

### Current App Deployment Order:
1. **Sealed Secrets Controller** (deploys first)
2. **Jenkins with SealedSecrets** (deploys after controller is ready)
3. **Controller automatically decrypts** SealedSecrets → regular Secrets
4. **Jenkins uses decrypted secrets** for admin password

### GitOps Flow:
```
Developer → Creates SealedSecret → Commits to Git → ArgoCD Syncs → 
Controller Decrypts → Jenkins Uses Secret
```

## 🛠 **Troubleshooting**

### Check if Controller is Running
```bash
kubectl get pods -n kube-system | grep sealed-secrets
kubectl logs -n kube-system deployment/sealed-secrets-controller
```

### Verify Secret Decryption
```bash
# Check if SealedSecret was decrypted
kubectl get sealedsecrets -n jenkins-dev
kubectl get secrets -n jenkins-dev | grep jenkins

# Check secret contents (base64 encoded)
kubectl get secret jenkins-dev-secrets -n jenkins-dev -o yaml
```

### Re-encrypt if Controller Changes
If you rebuild the cluster, the controller gets new keys:
```bash
# Get new public key
kubeseal --fetch-cert > public.pem

# Re-encrypt all secrets with new key
kubeseal --cert public.pem --format yaml < original-secret.yaml > new-sealedsecret.yaml
```

## 🔐 **Security Best Practices**

### Key Management
- Controller generates unique key pair per cluster
- Private key stays in cluster, never leaves
- Public key can be shared for encryption
- Keys rotate automatically (configurable)

### Access Control
- Only cluster controller can decrypt SealedSecrets
- SealedSecrets are tied to specific namespace/name
- Cannot be decrypted outside intended context

### Backup Strategy
```bash
# Backup controller private key (for disaster recovery)
kubectl get secret -n kube-system sealed-secrets-key -o yaml > sealed-secrets-key-backup.yaml

# Store backup securely (encrypted storage, vault, etc.)
```

## 📚 **Additional Resources**

- [Sealed Secrets GitHub](https://github.com/bitnami-labs/sealed-secrets)
- [Kubeseal CLI Docs](https://sealed-secrets.netlify.app/cli/)
- [GitOps with Sealed Secrets](https://blog.gitguardian.com/sealed-secrets/)

---

**Remember**: Never commit unencrypted secrets to Git! Always use SealedSecrets for sensitive data in GitOps workflows.
