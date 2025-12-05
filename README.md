# Jenkins GitOps with ArgoCD App-of-Apps Pattern

This repository contains a production-ready Jenkins deployment using GitOps principles with ArgoCD's App-of-Apps pattern and Kustomize for configuration management.

## 📁 Structure

```
├── root/
│   ├── app-of-apps.yaml          # Main ArgoCD Application
│   ├── kustomization.yaml        # Root Kustomize config
│   ├── jenkins-dev-app.yaml      # Jenkins Dev Application
│   └── jenkins-prod-app.yaml     # Jenkins Prod Application
└── apps/
    └── jenkins/
        ├── base/                 # Base Jenkins configuration
        │   ├── deployment.yaml
        │   ├── service.yaml
        │   ├── pvc.yaml
        │   ├── rbac.yaml
        │   ├── configmap.yaml    # JCasC configuration
        │   └── kustomization.yaml
        └── overlays/             # Environment-specific configs
            ├── dev/
            │   ├── kustomization.yaml
            │   ├── patches.yaml
            │   └── dev-jenkins.yaml
            └── prod/
                ├── kustomization.yaml
                ├── patches.yaml
                └── prod-jenkins.yaml
```

## 🚀 Features

### Jenkins Configuration as Code (JCasC)
- ✅ Automated Jenkins configuration
- ✅ Pre-configured users and security
- ✅ Kubernetes cloud integration
- ✅ Sample pipeline jobs
- ✅ Environment-specific configurations

### Environment Separation
- **Dev Environment**: Lower resources, development-focused jobs
- **Prod Environment**: High availability, production-grade resources

### GitOps Best Practices
- ✅ App-of-Apps pattern for scalability
- ✅ Kustomize base + overlays structure
- ✅ Environment-specific configurations
- ✅ Automated sync for dev, manual for prod

## 🛠 Deployment

### Prerequisites
- Kubernetes cluster
- ArgoCD installed
- Proper RBAC permissions

### 1. Deploy App-of-Apps
```bash
kubectl apply -f root/app-of-apps.yaml
```

### 2. Access Jenkins
After deployment, Jenkins will be available at:
- **Dev**: `http://your-cluster:32000` (NodePort)
- **Prod**: `http://your-cluster:32000` (NodePort)

### 3. Default Credentials
- **Dev**: admin/dev-admin123, developer/dev123
- **Prod**: admin/[from secret]

## 🔧 Customization

### Adding New Environments
1. Create new overlay directory: `apps/jenkins/overlays/staging/`
2. Copy and modify kustomization.yaml and patches
3. Add new ArgoCD Application in `root/`

### Modifying Jenkins Configuration
Edit the JCasC files in overlays:
- `dev-jenkins.yaml` for development
- `prod-jenkins.yaml` for production

### Resource Scaling
Modify resources in overlay patches:
```yaml
# apps/jenkins/overlays/prod/patches.yaml
resources:
  requests:
    memory: "2Gi"
    cpu: "1000m"
  limits:
    memory: "4Gi"
    cpu: "2000m"
```

## 🔐 Security Notes

### Production Security
- Use Kubernetes secrets for admin passwords
- Enable HTTPS with ingress controller
- Configure proper RBAC
- Regular security updates

### Secret Management
Create production secret:
```bash
kubectl create secret generic jenkins-secrets \
  --from-literal=admin-password=your-secure-password \
  -n jenkins-prod
```

## 📊 Monitoring

### Health Checks
Jenkins includes built-in health endpoints:
- Health: `http://jenkins:8080/login`
- Metrics: Available via JMX

### ArgoCD Monitoring
Monitor application sync status in ArgoCD UI:
- App-of-Apps status
- Individual application health
- Sync history and events

## 🔄 CI/CD Integration

### Pipeline as Code
Jenkins includes sample pipelines configured via JCasC:
- Development pipeline with basic stages
- Production pipeline with security scans

### Kubernetes Integration
- Dynamic agent provisioning
- Kubernetes-native builds
- Container-based workflows

## 📚 Additional Resources

- [Jenkins Configuration as Code](https://github.com/jenkinsci/configuration-as-code-plugin)
- [ArgoCD App of Apps](https://argo-cd.readthedocs.io/en/stable/operator-manual/cluster-bootstrapping/)
- [Kustomize Documentation](https://kustomize.io/)

## 🤝 Contributing

1. Fork the repository
2. Create feature branch
3. Make changes to appropriate overlay
4. Test in development environment
5. Submit pull request

---

**Note**: Remember to update the repository URL in ArgoCD applications to match your actual Git repository.
