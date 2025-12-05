# Jenkins Dynamic Kubernetes Agents

This Jenkins setup is configured to automatically provision Kubernetes agents on-demand. **No manual agent setup required!**

## 🚀 **How It Works**

Jenkins will **automatically create pods** when builds are triggered:

1. **Job starts** → Jenkins checks available agents
2. **No agents available** → Jenkins creates a new pod
3. **Pod starts** → Agent connects to Jenkins
4. **Job runs** → Work is executed in the pod
5. **Job completes** → Pod is automatically deleted

## 🎯 **Available Agent Types**

### **Default Agent** (`label: default`)
- **Use case**: Lightweight tasks, shell scripts, basic operations
- **Resources**: 100m CPU, 256Mi RAM → 500m CPU, 1Gi RAM
- **Contains**: Jenkins agent + JDK 17

### **Docker Agent** (`label: docker`)
- **Use case**: Building Docker images, container operations
- **Resources**: 300m CPU, 768Mi RAM → 1.5 CPU, 3Gi RAM
- **Contains**: Jenkins agent + Docker-in-Docker
- **Special**: Privileged mode for Docker builds

### **Maven/Java Agent** (`label: maven`)
- **Use case**: Java applications, Maven/Gradle builds
- **Resources**: 300m CPU, 768Mi RAM → 1.5 CPU, 3Gi RAM
- **Contains**: Jenkins agent + Maven 3.9 + OpenJDK 17

### **Node.js Agent** (`label: nodejs`)
- **Use case**: Frontend builds, npm/yarn operations
- **Resources**: 300m CPU, 768Mi RAM → 1.5 CPU, 3Gi RAM
- **Contains**: Jenkins agent + Node.js 18

## 💡 **Pipeline Examples**

### **Using Specific Agents**
```groovy
pipeline {
    agent none
    stages {
        stage('Build Java') {
            agent { label 'maven' }
            steps {
                container('maven') {
                    sh 'mvn clean compile'
                }
            }
        }
        stage('Build Frontend') {
            agent { label 'nodejs' }
            steps {
                container('nodejs') {
                    sh 'npm install && npm run build'
                }
            }
        }
    }
}
```

### **Docker Build Example**
```groovy
pipeline {
    agent { label 'docker' }
    stages {
        stage('Build Image') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp .'
                    sh 'docker push myapp:latest'
                }
            }
        }
    }
}
```

### **Parallel Builds with Different Agents**
```groovy
pipeline {
    agent none
    stages {
        stage('Parallel Builds') {
            parallel {
                stage('Backend') {
                    agent { label 'maven' }
                    steps {
                        container('maven') {
                            sh 'mvn test'
                        }
                    }
                }
                stage('Frontend') {
                    agent { label 'nodejs' }
                    steps {
                        container('nodejs') {
                            sh 'npm test'
                        }
                    }
                }
            }
        }
    }
}
```

## ⚙️ **Configuration Details**

### **Agent Lifecycle**
- **Idle timeout**: 5 minutes (agents auto-delete when idle)
- **Pod retention**: Never (pods deleted immediately after job)
- **Max agents**: 50 concurrent pods
- **Connection timeout**: 10 seconds

### **Resource Management**
- All agents have **resource requests** and **limits**
- Kubernetes scheduler places pods based on available resources
- Failed pods are automatically recreated

### **Security**
- Agents run with `jenkins` ServiceAccount
- Proper RBAC permissions for pod management
- No privileged access except for Docker agent

## 🔍 **Monitoring Agents**

### **Via Jenkins UI**
- Go to **Manage Jenkins** → **Manage Nodes and Clouds**
- View **Kubernetes** cloud configuration
- See active agents and their status

### **Via kubectl**
```bash
# List Jenkins agent pods
kubectl get pods -l jenkins=slave

# Watch agents being created/destroyed
kubectl get pods -l jenkins=slave -w

# Check agent logs
kubectl logs -f <pod-name>
```

## 🛠 **Troubleshooting**

### **Agents Not Starting**
1. Check Jenkins logs: **Manage Jenkins** → **System Log**
2. Verify RBAC: `kubectl auth can-i create pods --as=system:serviceaccount:default:jenkins`
3. Check resource availability: `kubectl describe nodes`

### **Build Failures**
1. Check if correct agent label is used
2. Verify container has required tools
3. Check resource limits aren't too low

### **Slow Agent Startup**
1. Enable image pre-pulling: `alwaysPullImage: false`
2. Use smaller base images
3. Increase resource requests

## 📈 **Scaling**

The setup automatically scales based on demand:
- **Light load**: 0-2 agents running
- **Heavy load**: Up to 50 agents (configurable)
- **No waste**: Agents destroyed when not needed

This gives you **infinite build capacity** without managing physical agents! 🎉
