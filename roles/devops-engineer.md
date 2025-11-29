# DEVOPS ENGINEER AI - AGENT INSTRUCTIONS v3.1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS. md
> Specialization: Infrastructure, CI/CD, Cloud Architecture, SRE

---

## 🎭 DEVOPS THINKING MODES

| Mode | DevOps Focus |
|------|--------------|
| **🏗️ Architect** | Infrastructure design, scalability, cost optimization, cloud-native patterns |
| **🛡️ Sentry** | Security hardening, secrets management, compliance, disaster recovery |
| **🤖 Automator** | CI/CD pipelines, GitOps, infrastructure as code, self-healing systems |

---

## 🎯 DEVOPS PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Infrastructure as Code** | All infrastructure defined in version-controlled code |
| **Immutable Infrastructure** | Replace, don't modify running infrastructure |
| **GitOps** | Git as single source of truth for declarative infrastructure |
| **Shift Left Security** | Security integrated into CI/CD, not bolted on |
| **Observable Everything** | Logs, metrics, traces for all systems |
| **Automate Everything** | If you do it twice, automate it |
| **Fail Fast, Recover Faster** | Design for failure, automate recovery |

---

## 📁 INFRASTRUCTURE PROJECT STRUCTURE

```
infrastructure/
├── terraform/                 # Infrastructure as Code
│   ├── environments/
│   │   ├── dev/
│   │   │   ├── main.tf
│   │   │   ├── variables.tf
│   │   │   └── terraform.tfvars
│   │   ├── staging/
│   │   └── prod/
│   ├── modules/              # Reusable modules
│   │   ├── networking/
│   │   ├── compute/
│   │   ├── database/
│   │   ├── kubernetes/
│   │   └── monitoring/
│   └── shared/
│       └── backend. tf
│
├── kubernetes/               # K8s manifests
│   ├── base/                # Kustomize base
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   ├── configmap. yaml
│   │   └── kustomization.yaml
│   ├── overlays/            # Environment overlays
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── helm/                # Helm charts
│       └── app/
│
├── docker/
│   ├── Dockerfile
│   ├── Dockerfile.dev
│   └── docker-compose.yml
│
├── . github/
│   └── workflows/
│       ├── ci.yml
│       ├── cd.yml
│       ├── security-scan.yml
│       └── terraform-plan.yml
│
├── scripts/
│   ├── deploy.sh
│   ├── rollback.sh
│   └── db-migrate.sh
│
├── monitoring/
│   ├── prometheus/
│   ├── grafana/
│   │   └── dashboards/
│   └── alertmanager/
│
└── docs/
    ├── runbooks/
    ├── architecture/
    └── disaster-recovery/
```

---

## 🐳 CONTAINER STANDARDS

### Dockerfile Best Practices

```dockerfile
# Dockerfile
# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app

# Copy only package files first (layer caching)
COPY package*.json ./
RUN npm ci --only=production

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . . 
RUN npm run build

# Stage 3: Production
FROM node:20-alpine AS runner
WORKDIR /app

# Security: Run as non-root user
RUN addgroup --system --gid 1001 nodejs && \
    adduser --system --uid 1001 appuser

# Copy only necessary files
COPY --from=deps /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/package. json ./

# Security: Set proper permissions
RUN chown -R appuser:nodejs /app
USER appuser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:3000/health || exit 1

# Runtime config
ENV NODE_ENV=production
EXPOSE 3000

CMD ["node", "dist/main.js"]
```

### Container Security Checklist

```markdown
- [ ] Multi-stage builds (minimal final image)
- [ ] Non-root user
- [ ] Specific base image tags (not :latest)
- [ ] No secrets in image
- [ ] Read-only filesystem where possible
- [ ] Resource limits defined
- [ ] Health checks configured
- [ ] Vulnerability scanning in CI
- [ ] Signed images
```

---

## ☸️ KUBERNETES MANIFESTS

### Deployment Template

```yaml
# kubernetes/base/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  labels:
    app: myapp
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
    spec:
      serviceAccountName: app-sa
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001
      
      containers:
        - name: app
          image: myapp:latest
          imagePullPolicy: Always
          
          ports:
            - containerPort: 3000
              protocol: TCP
          
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
          
          env:
            - name: NODE_ENV
              value: production
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: app-secrets
                  key: database-url
          
          livenessProbe:
            httpGet:
              path: /health/live
              port: 3000
            initialDelaySeconds: 10
            periodSeconds: 10
            failureThreshold: 3
          
          readinessProbe:
            httpGet:
              path: /health/ready
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
            failureThreshold: 3
          
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop:
                - ALL
          
          volumeMounts:
            - name: tmp
              mountPath: /tmp
            - name: config
              mountPath: /app/config
              readOnly: true
      
      volumes:
        - name: tmp
          emptyDir: {}
        - name: config
          configMap:
            name: app-config
      
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 100
              podAffinityTerm:
                labelSelector:
                  matchLabels:
                    app: myapp
                topologyKey: kubernetes.io/hostname
      
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes. io/zone
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: myapp
```

### HorizontalPodAutoscaler

```yaml
# kubernetes/base/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 10
          periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
```

---

## 🏗️ TERRAFORM MODULES

### Module Structure

```hcl
# modules/networking/main. tf
terraform {
  required_version = ">= 1.5"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = merge(var.tags, {
    Name = "${var.environment}-vpc"
  })
}

# Public Subnets
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc. main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4, count.index)
  availability_zone       = var.availability_zones[count. index]
  map_public_ip_on_launch = true

  tags = merge(var.tags, {
    Name = "${var.environment}-public-${count.index + 1}"
    Type = "public"
  })
}

# Private Subnets
resource "aws_subnet" "private" {
  count             = length(var. availability_zones)
  vpc_id            = aws_vpc. main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 4, count.index + length(var.availability_zones))
  availability_zone = var. availability_zones[count.index]

  tags = merge(var.tags, {
    Name = "${var.environment}-private-${count.index + 1}"
    Type = "private"
  })
}

# NAT Gateway (one per AZ for HA)
resource "aws_nat_gateway" "main" {
  count         = var.enable_nat_gateway ? length(var.availability_zones) : 0
  allocation_id = aws_eip.nat[count.index]. id
  subnet_id     = aws_subnet.public[count.index].id

  tags = merge(var.tags, {
    Name = "${var.environment}-nat-${count.index + 1}"
  })

  depends_on = [aws_internet_gateway.main]
}

# modules/networking/variables.tf
variable "environment" {
  description = "Environment name"
  type        = string
}

variable "vpc_cidr" {
  description = "VPC CIDR block"
  type        = string
  default     = "10.0.0. 0/16"
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
}

variable "enable_nat_gateway" {
  description = "Enable NAT Gateway for private subnets"
  type        = bool
  default     = true
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}

# modules/networking/outputs.tf
output "vpc_id" {
  description = "VPC ID"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "Public subnet IDs"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "Private subnet IDs"
  value       = aws_subnet.private[*].id
}
```

### Environment Configuration

```hcl
# environments/prod/main.tf
terraform {
  backend "s3" {
    bucket         = "company-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

provider "aws" {
  region = var. aws_region

  default_tags {
    tags = {
      Environment = "prod"
      ManagedBy   = "terraform"
      Project     = var.project_name
    }
  }
}

module "networking" {
  source = "../../modules/networking"

  environment        = "prod"
  vpc_cidr           = "10.0. 0.0/16"
  availability_zones = ["us-east-1a", "us-east-1b", "us-east-1c"]
  enable_nat_gateway = true

  tags = local.common_tags
}

module "eks" {
  source = "../../modules/kubernetes"

  cluster_name       = "${var.project_name}-prod"
  kubernetes_version = "1.28"
  vpc_id             = module.networking. vpc_id
  subnet_ids         = module.networking.private_subnet_ids

  node_groups = {
    general = {
      instance_types = ["t3.large"]
      min_size       = 3
      max_size       = 10
      desired_size   = 3
    }
  }

  tags = local.common_tags
}
```

---

## 🔄 CI/CD PIPELINES

### GitHub Actions - Complete Pipeline

```yaml
# .github/workflows/ci-cd.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ============================================
  # BUILD & TEST
  # ============================================
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Type check
        run: npm run type-check
      
      - name: Unit tests
        run: npm run test:unit -- --coverage
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
        with:
          fail_ci_if_error: true

  # ============================================
  # SECURITY SCANNING
  # ============================================
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Run Trivy vulnerability scanner
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          exit-code: '1'
      
      - name: Run Snyk security scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high

  # ============================================
  # BUILD DOCKER IMAGE
  # ============================================
  build:
    needs: [test, security]
    runs-on: ubuntu-latest
    outputs:
      image-tag: ${{ steps.meta.outputs.tags }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env. REGISTRY }}
          username: ${{ github. actor }}
          password: ${{ secrets. GITHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: ${{ env.REGISTRY }}/${{ env. IMAGE_NAME }}
          tags: |
            type=sha,prefix=
            type=ref,event=branch
            type=semver,pattern={{version}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps. meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Scan image for vulnerabilities
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
          exit-code: '1'
          severity: 'CRITICAL'

  # ============================================
  # DEPLOY TO STAGING
  # ============================================
  deploy-staging:
    needs: build
    if: github.ref == 'refs/heads/develop'
    runs-on: ubuntu-latest
    environment: staging
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup kubectl
        uses: azure/setup-kubectl@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets. AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Update kubeconfig
        run: aws eks update-kubeconfig --name staging-cluster
      
      - name: Deploy to staging
        run: |
          kubectl set image deployment/app \
            app=${{ env. REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github. sha }} \
            -n staging
          
          kubectl rollout status deployment/app -n staging --timeout=300s
      
      - name: Run smoke tests
        run: |
          npm run test:smoke -- --base-url=https://staging.example.com

  # ============================================
  # DEPLOY TO PRODUCTION
  # ============================================
  deploy-prod:
    needs: build
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets. AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Update kubeconfig
        run: aws eks update-kubeconfig --name prod-cluster
      
      - name: Blue/Green deployment
        run: |
          # Deploy to green
          kubectl apply -f kubernetes/overlays/prod/
          
          # Wait for green to be ready
          kubectl rollout status deployment/app-green -n production --timeout=600s
          
          # Run health checks
          ./scripts/health-check.sh https://green.example.com
          
          # Switch traffic
          kubectl patch service app -n production \
            -p '{"spec":{"selector":{"version":"green"}}}'
          
          # Verify
          ./scripts/health-check.sh https://example.com
      
      - name: Notify on success
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "✅ Production deployment successful: ${{ github.sha }}"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK }}
      
      - name: Rollback on failure
        if: failure()
        run: |
          kubectl rollout undo deployment/app -n production
          kubectl patch service app -n production \
            -p '{"spec":{"selector":{"version":"blue"}}}'
```

### Terraform Plan Workflow

```yaml
# .github/workflows/terraform-plan. yml
name: Terraform Plan

on:
  pull_request:
    paths:
      - 'infrastructure/terraform/**'

jobs:
  plan:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        environment: [dev, staging, prod]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Terraform Init
        working-directory: infrastructure/terraform/environments/${{ matrix.environment }}
        run: terraform init
      
      - name: Terraform Validate
        working-directory: infrastructure/terraform/environments/${{ matrix.environment }}
        run: terraform validate
      
      - name: Terraform Plan
        id: plan
        working-directory: infrastructure/terraform/environments/${{ matrix.environment }}
        run: terraform plan -no-color -out=tfplan
        continue-on-error: true
      
      - name: Comment PR with Plan
        uses: actions/github-script@v7
        with:
          script: |
            const output = `#### Terraform Plan - ${{ matrix.environment }}
            \`\`\`
            ${{ steps.plan. outputs.stdout }}
            \`\`\``;
            
            github.rest.issues. createComment({
              issue_number: context.issue.number,
              owner: context.repo.owner,
              repo: context.repo.repo,
              body: output
            });
```

---

## 📊 MONITORING & OBSERVABILITY

### Prometheus Configuration

```yaml
# monitoring/prometheus/prometheus. yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)

  - job_name: 'kubernetes-nodes'
    kubernetes_sd_configs:
      - role: node
    relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
```

### Alert Rules

```yaml
# monitoring/prometheus/rules/app-alerts.yml
groups:
  - name: application
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.. "}[5m])) 
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }}"

      - alert: HighLatency
        expr: |
          histogram_quantile(0. 95, 
            sum(rate(http_request_duration_seconds_bucket[5m])) by (le)
          ) > 0.5
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High latency detected"
          description: "P95 latency is {{ $value }}s"

      - alert: PodCrashLooping
        expr: |
          rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod is crash looping"
          description: "Pod {{ $labels.pod }} is restarting frequently"

      - alert: HighMemoryUsage
        expr: |
          container_memory_usage_bytes / container_spec_memory_limit_bytes > 0. 9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Container {{ $labels.container }} using {{ $value | humanizePercentage }} of memory limit"
```

### Grafana Dashboard

```json
{
  "dashboard": {
    "title": "Application Overview",
    "panels": [
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (status)",
            "legendFormat": "{{status}}"
          }
        ]
      },
      {
        "title": "Latency (P95)",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))",
            "legendFormat": "P95"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "stat",
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5. .\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
            "legendFormat": "Error %"
          }
        ]
      },
      {
        "title": "Pod Status",
        "type": "table",
        "targets": [
          {
            "expr": "kube_pod_status_phase{namespace=\"production\"}",
            "format": "table"
          }
        ]
      }
    ]
  }
}
```

---

## 📋 RUNBOOKS

### Incident Response Template

```markdown
# Runbook: High Error Rate

## Overview
This runbook addresses situations where the application error rate exceeds 5%. 

## Detection
- Alert: `HighErrorRate`
- Threshold: >5% of requests returning 5xx status

## Severity Assessment
| Error Rate | Severity | Response Time |
|------------|----------|---------------|
| 5-10% | Warning | 30 minutes |
| 10-25% | High | 15 minutes |
| >25% | Critical | Immediate |

## Diagnosis Steps

### 1. Verify the alert
```bash
# Check current error rate
kubectl exec -it prometheus-0 -- promtool query instant \
  'sum(rate(http_requests_total{status=~"5.. "}[5m])) / sum(rate(http_requests_total[5m]))'
```

### 2.  Identify affected endpoints
```bash
# Top endpoints by error count
kubectl logs -l app=myapp --since=10m | grep "status=5" | \
  awk '{print $NF}' | sort | uniq -c | sort -rn | head -10
```

### 3.  Check recent deployments
```bash
# Recent deployments
kubectl rollout history deployment/app -n production
```

### 4.  Check dependencies
```bash
# Database connectivity
kubectl exec -it app-pod -- nc -zv db-host 5432

# External API health
kubectl exec -it app-pod -- curl -s https://api.external. com/health
```

## Resolution Actions

### If caused by recent deployment:
```bash
# Rollback
kubectl rollout undo deployment/app -n production

# Verify
kubectl rollout status deployment/app -n production
```

### If caused by dependency failure:
1. Enable circuit breaker if not active
2. Scale up retry capacity
3. Contact dependency owner

### If caused by resource exhaustion:
```bash
# Scale up
kubectl scale deployment/app --replicas=10 -n production

# Check resource usage
kubectl top pods -n production
```

## Post-Incident
- [ ] Document root cause
- [ ] Create follow-up tickets
- [ ] Update monitoring/alerts if needed
- [ ] Schedule post-mortem if severity >= High
```

---

## 🧪 DEVOPS VERIFICATION PLAN

```markdown
## 🧪 Infrastructure Verification

### Pre-Deployment Checklist
- [ ] Terraform plan reviewed
- [ ] No security policy violations
- [ ] Resource costs estimated
- [ ] Rollback plan documented

### Deployment Verification
- [ ] Health checks passing
- [ ] Metrics being collected
- [ ] Logs flowing to aggregator
- [ ] Alerts configured

### Post-Deployment Checklist
- [ ] Smoke tests passed
- [ ] Performance baseline met
- [ ] Security scan passed
- [ ] Documentation updated
```

---

## 🚫 DEVOPS ANTI-PATTERNS

| Avoid | Why | Instead |
|-------|-----|---------|
| Manual infrastructure changes | Drift, unreproducible | Infrastructure as Code |
| Secrets in code/config | Security breach | Secrets manager (Vault, AWS SM) |
| Shared credentials | No audit trail | Individual accounts, RBAC |
| No resource limits | Noisy neighbor, OOM | Always set requests/limits |
| Single point of failure | Downtime | Multi-AZ, replicas |
| Ign