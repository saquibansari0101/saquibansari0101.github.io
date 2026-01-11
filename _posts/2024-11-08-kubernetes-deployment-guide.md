---
layout: post
title: Deploying Microservices on Kubernetes - A Practical Guide
---

Kubernetes has become the de facto standard for container orchestration, and for good reason. Here's what I've learned deploying and operating microservices on Azure Kubernetes Service (AKS) in production.

## Why Kubernetes?

Before Kubernetes, deploying and scaling microservices was challenging:
- Manual deployment processes were error-prone
- Scaling required manual intervention
- Service discovery was complex
- Rolling updates caused downtime

Kubernetes solved all these problems and more.

## Our Kubernetes Setup

At Infosys, we deploy all microservices on Azure Kubernetes Service (AKS) with:
- **Multiple namespaces** for environment isolation (dev, staging, prod)
- **Helm charts** for templated deployments
- **Azure DevOps** for CI/CD pipelines
- **Prometheus & Grafana** for monitoring
- **ELK stack** for centralized logging

## Key Kubernetes Concepts I Use Daily

### 1. Deployments
Deployments manage the desired state of your application:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: data-pipeline-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: data-pipeline
  template:
    metadata:
      labels:
        app: data-pipeline
    spec:
      containers:
      - name: data-pipeline
        image: myregistry.azurecr.io/data-pipeline:v1.2.0
        ports:
        - containerPort: 8080
        env:
        - name: KAFKA_BOOTSTRAP_SERVERS
          valueFrom:
            configMapKeyRef:
              name: kafka-config
              key: bootstrap.servers
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 5
```

### 2. Services
Services provide stable networking for pods:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: data-pipeline-service
spec:
  selector:
    app: data-pipeline
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP
```

### 3. ConfigMaps and Secrets
Externalize configuration from container images:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kafka-config
data:
  bootstrap.servers: "kafka-broker-1:9092,kafka-broker-2:9092"
  
---
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=  # base64 encoded
```

### 4. Horizontal Pod Autoscaling
Automatically scale based on metrics:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: data-pipeline-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: data-pipeline-service
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

## CI/CD Pipeline with Azure DevOps

Our deployment pipeline:

1. **Build Stage**
   - Run unit tests
   - Build Docker image
   - Push to Azure Container Registry

2. **Deploy Stage**
   - Update Helm values
   - Deploy to Kubernetes using `kubectl` or Helm
   - Run smoke tests
   - Monitor rollout status

```yaml
# Azure DevOps pipeline snippet
- task: Kubernetes@1
  displayName: Deploy to AKS
  inputs:
    connectionType: 'Azure Resource Manager'
    azureSubscriptionEndpoint: '$(azureSubscription)'
    azureResourceGroup: '$(resourceGroup)'
    kubernetesCluster: '$(aksCluster)'
    command: 'apply'
    arguments: '-f k8s/deployment.yaml'
```

## Best Practices I Follow

### 1. Resource Limits
Always set resource requests and limits to:
- Prevent resource starvation
- Enable efficient scheduling
- Avoid OOM kills

### 2. Health Checks
Implement liveness and readiness probes:
- **Liveness**: Restart unhealthy containers
- **Readiness**: Remove pods from load balancing when not ready

### 3. Rolling Updates
Use rolling update strategy for zero-downtime deployments:

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1
    maxUnavailable: 0
```

### 4. Pod Disruption Budgets
Ensure availability during cluster maintenance:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: data-pipeline-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: data-pipeline
```

### 5. Network Policies
Implement security at the network level:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: data-pipeline-netpol
spec:
  podSelector:
    matchLabels:
      app: data-pipeline
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-gateway
```

## Monitoring and Troubleshooting

### Common Commands I Use

```bash
# Check pod status
kubectl get pods -n production

# View logs
kubectl logs -f pod-name -n production

# Describe pod for events
kubectl describe pod pod-name -n production

# Execute commands in pod
kubectl exec -it pod-name -n production -- /bin/bash

# Port forward for local testing
kubectl port-forward pod-name 8080:8080 -n production

# Check resource usage
kubectl top pods -n production
```

### Debugging Tips

1. **Pod not starting**: Check events with `kubectl describe pod`
2. **CrashLoopBackOff**: Check logs and liveness probe configuration
3. **ImagePullBackOff**: Verify image name and registry credentials
4. **High memory usage**: Review resource limits and application memory leaks

## Lessons Learned

**Start Simple**  
Don't over-engineer. Start with basic deployments and services, then add complexity as needed.

**Automate Everything**  
Manual `kubectl` commands don't scale. Use CI/CD pipelines and GitOps.

**Monitor Proactively**  
Set up alerts for pod restarts, high resource usage, and failed deployments.

**Plan for Failure**  
Design for pod failures, node failures, and availability zone failures.

**Security First**  
Use RBAC, network policies, and secrets management from day one.

## Conclusion

Kubernetes has transformed how we deploy and operate microservices. While there's a learning curve, the benefits—scalability, resilience, and automation—are well worth it.

If you're starting with Kubernetes, focus on understanding the core concepts, start with simple deployments, and gradually adopt advanced features as your needs grow.
