# AiGentForce-V2 Deployment Guide

## Overview

This guide covers the deployment of AiGentForce-V2 across development, staging, and production environments.

## Pre-Deployment Checklist

- [ ] All tests passing (unit, integration, e2e)
- [ ] Security scanning completed
- [ ] Database migrations reviewed
- [ ] Environment variables configured
- [ ] API keys and credentials secured
- [ ] Documentation updated
- [ ] Code reviewed and approved
- [ ] Performance benchmarks acceptable

## Environment Setup

### Prerequisites

- Node.js 16+ or Python 3.9+
- PostgreSQL 12+
- Redis 6+
- Docker and Docker Compose
- Kubernetes cluster (for production)
- AWS/Azure/GCP credentials

### Development Environment

```bash
# Clone repository
git clone https://github.com/tosodo/AiGentForce-V2.git
cd AiGentForce-V2

# Install dependencies
npm install

# Setup local database
cp .env.example .env.local
npx prisma migrate dev

# Start development server
npm run dev
```

### Staging Environment

```bash
# Build Docker image
docker build -t aigentforce-v2:latest .

# Push to container registry
docker tag aigentforce-v2:latest gcr.io/project-id/aigentforce-v2:latest
docker push gcr.io/project-id/aigentforce-v2:latest

# Deploy to Kubernetes staging cluster
kubectl --context=staging apply -f k8s/deployment.yaml
```

### Production Environment

```bash
# Build and push Docker image with version tag
docker build -t aigentforce-v2:v2.0.0 .
docker push gcr.io/project-id/aigentforce-v2:v2.0.0

# Deploy to production cluster
kubectl --context=production apply -f k8s/deployment.yaml

# Verify deployment
kubectl rollout status deployment/aigentforce-v2 -n production
```

## Database Deployment

### Migrations

```bash
# Create migration
npx prisma migrate dev --name add_new_feature

# Run migration (production)
npx prisma migrate deploy

# Rollback migration
npx prisma migrate resolve --rolled-back <migration-name>
```

### Backup and Restore

```bash
# Backup database
pg_dump -U postgres aigentforce_db > backup.sql

# Restore database
psql -U postgres aigentforce_db < backup.sql

# Create backup schedule (cron job)
0 2 * * * /scripts/backup.sh
```

## Configuration Management

### Environment Variables

**Required Variables:**
```
DATABASE_URL=postgresql://user:password@host/db
REDIS_URL=redis://localhost:6379
OPENAI_API_KEY=sk-...
LANGCHAIN_API_KEY=...
N8N_WEBHOOK_URL=https://n8n.example.com
AIGENTFORCE_IO_API_KEY=...
JWT_SECRET=your-secret-key
NODE_ENV=production
```

### Secrets Management

```bash
# Using AWS Secrets Manager
aws secretsmanager create-secret --name aigentforce/api-keys --secret-string file://secrets.json

# Using Google Secret Manager
gcloud secrets create aigentforce-api-keys --data-file=secrets.json

# Using HashiCorp Vault
vault kv put secret/aigentforce @secrets.json
```

## Deployment Strategies

### Rolling Deployment

```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aigentforce-v2
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  replicas: 3
```

### Blue-Green Deployment

```bash
# Deploy new version (green)
kubectl apply -f k8s/deployment-green.yaml

# Wait for health checks
sleep 60

# Switch traffic to green
kubectl patch service aigentforce -p '{"spec":{"selector":{"version":"green"}}}'

# Keep blue running for rollback
kubectl delete -f k8s/deployment-blue.yaml
```

### Canary Deployment

```yaml
# Route 10% of traffic to new version
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: aigentforce
spec:
  hosts:
    - aigentforce
  http:
    - match:
        - uri:
            prefix: /
      route:
        - destination:
            host: aigentforce
            subset: v1
          weight: 90
        - destination:
            host: aigentforce
            subset: v2
          weight: 10
```

## Health Checks and Monitoring

### Readiness Probe

```yaml
readinessProbe:
  httpGet:
    path: /health/ready
    port: 3000
  initialDelaySeconds: 10
  periodSeconds: 5
```

### Liveness Probe

```yaml
livenessProbe:
  httpGet:
    path: /health/live
    port: 3000
  initialDelaySeconds: 30
  periodSeconds: 10
```

## Scaling

### Horizontal Pod Autoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: aigentforce-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: aigentforce-v2
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

## Logging and Monitoring

### Centralized Logging

```bash
# ELK Stack setup
docker-compose up -d elasticsearch kibana logstash

# Application logs
curl -X PUT http://localhost:9200/aigentforce-logs
```

### Metrics Collection

```bash
# Prometheus scrape config
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
data:
  prometheus.yml: |
    scrape_configs:
      - job_name: 'aigentforce'
        static_configs:
          - targets: ['localhost:9090']
```

## Rollback Procedures

### Kubernetes Rollback

```bash
# View rollout history
kubectl rollout history deployment/aigentforce-v2

# Rollback to previous version
kubectl rollout undo deployment/aigentforce-v2

# Rollback to specific revision
kubectl rollout undo deployment/aigentforce-v2 --to-revision=2
```

### Database Rollback

```bash
# Revert migration
npx prisma migrate resolve --rolled-back <migration-id>

# Restore from backup
psql -U postgres aigentforce_db < backup_$(date +%Y%m%d).sql
```

## Post-Deployment

### Smoke Tests

```bash
# Run smoke test suite
npm run test:smoke

# Check critical endpoints
curl -X GET https://api.example.com/health
curl -X GET https://api.example.com/v1/status
```

### Performance Validation

```bash
# Run load tests
artillery run load-test.yml

# Monitor performance metrics
watch kubectl top nodes
watch kubectl top pods
```

## Troubleshooting

### Pod Not Starting

```bash
# Check pod status
kubectl describe pod aigentforce-v2-xxxx

# Check logs
kubectl logs aigentforce-v2-xxxx
kubectl logs aigentforce-v2-xxxx --previous
```

### Connection Issues

```bash
# Test database connection
kubectl exec aigentforce-v2-xxxx -- psql $DATABASE_URL -c "SELECT 1"

# Test Redis connection
kubectl exec aigentforce-v2-xxxx -- redis-cli ping
```

## Disaster Recovery

### Backup Schedule

```bash
# Daily backup at 2 AM
0 2 * * * /scripts/backup.sh

# Weekly full backup
0 3 * * 0 /scripts/backup-full.sh

# Monthly snapshot
0 4 1 * * /scripts/snapshot.sh
```

### Recovery Testing

- Monthly recovery drills
- Backup verification
- RTO/RPO validation
- Documentation updates

## Security in Deployment

### Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: aigentforce-policy
spec:
  podSelector:
    matchLabels:
      app: aigentforce
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - namespaceSelector:
            matchLabels:
              name: ingress
```

### RBAC Configuration

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: aigentforce-role
rules:
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list"]
```

## Support and Contact

For deployment issues:
- Check logs: `kubectl logs -f deployment/aigentforce-v2`
- Review status: `kubectl get all -n production`
- Contact: [deployment-support@aigentforce.io](mailto:deployment-support@aigentforce.io)

---

For architecture details, see [ARCHITECTURE.md](ARCHITECTURE.md)  
For integration setup, see [INTEGRATIONS.md](INTEGRATIONS.md)
