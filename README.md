# Alta Entrega - Infrastructure

**Infrastructure as Code | Docker Compose, Kubernetes, Terraform, Configs**

---

## 🎯 Visão Geral

Repositório de infraestrutura para o Alta Entrega, contendo:

- **Docker Compose:** Ambiente de desenvolvimento completo (30+ services)
- **Kubernetes:** Manifests para produção
- **Terraform:** Infrastructure as Code para cloud providers
- **Configs:** Configurações para Redpanda, Temporal, Grafana, Prometheus, etc.

---

## 🏗️ Estrutura

```
alta-entrega-infra/
├── docker-compose/
│   ├── docker-compose.yml           # Core services
│   ├── docker-compose.dev.yml       # Development overrides
│   ├── docker-compose.observability.yml
│   └── .env.example
├── kubernetes/
│   ├── base/                        # Base manifests
│   ├── overlays/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── kustomization.yaml
├── terraform/
│   ├── modules/
│   │   ├── vpc/
│   │   ├── eks/
│   │   ├── rds/
│   │   └── redis/
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── production/
│   └── main.tf
├── configs/
│   ├── redpanda/
│   ├── temporal/
│   ├── grafana/
│   ├── prometheus/
│   └── traefik/
└── scripts/
    ├── setup-dev.sh
    ├── backup.sh
    └── restore.sh
```

---

## 🚀 Stack de Infraestrutura

### Core Data & Performance
- **PostgreSQL 16** (OLTP) + PgBouncer
- **Redis Cluster** (cache, rate limit)
- **ClickHouse** (OLAP, analytics)

### Event Streaming
- **Redpanda** (Kafka-compatible event bus)
- **Redpanda Console** (UI management)
- **KafkaJS** (Node.js client)

### Workflows
- **Temporal Server** (durable workflows)
- **Temporal UI** (workflow monitoring)

### Observability
- **OpenTelemetry Collector**
- **Prometheus** (metrics)
- **Grafana** (dashboards)
- **Loki** (logs)
- **Tempo** (traces)

### Feature Management
- **Unleash** (feature flags)

### Search & Storage
- **Meilisearch** (full-text search)
- **MinIO** (S3-compatible object storage)

### Automation
- **n8n** (workflow automation)

### Auth & Policy
- **Supabase** (Auth + RLS - MVP)
- **Keycloak** (SSO/RBAC - Enterprise)
- **Open Policy Agent** (policy engine)

### Ingress
- **Traefik** (reverse proxy, TLS, routing)

---

## 🔧 Setup - Desenvolvimento

### Pré-requisitos
- Docker 24+
- Docker Compose v2
- 16GB+ RAM
- 50GB+ disco disponível

### Iniciar Ambiente Completo

```bash
# Clone o repositório
git clone https://github.com/vinicsantana/alta-entrega-infra.git
cd alta-entrega-infra

# Configure variáveis de ambiente
cp docker-compose/.env.example docker-compose/.env

# Inicie todos os serviços (30+ containers)
cd docker-compose
docker-compose up -d

# Verifique o status
docker-compose ps

# Logs de um serviço específico
docker-compose logs -f redpanda
```

### Iniciar Apenas Core Services

```bash
# PostgreSQL + Redis + Redpanda + Temporal
docker-compose up -d postgres redis redpanda temporal
```

### Acessar UIs

| Service | URL | Credentials |
|---------|-----|-------------|
| Redpanda Console | http://localhost:8080 | - |
| Temporal UI | http://localhost:8081 | - |
| Grafana | http://localhost:3001 | admin/admin |
| Prometheus | http://localhost:9090 | - |
| MinIO Console | http://localhost:9001 | minioadmin/minioadmin |
| n8n | http://localhost:5678 | - |
| Unleash | http://localhost:4242 | admin/unleash4all |

---

## ☸️ Kubernetes - Produção

### Pré-requisitos
- Kubernetes 1.28+
- kubectl
- Kustomize
- Helm 3+

### Deploy

```bash
cd kubernetes

# Deploy base + staging
kubectl apply -k overlays/staging

# Deploy production
kubectl apply -k overlays/production

# Verificar pods
kubectl get pods -n alta-entrega
```

### Namespaces

- `alta-entrega-dev`
- `alta-entrega-staging`
- `alta-entrega-prod`
- `alta-entrega-observability`
- `alta-entrega-events` (Redpanda)

---

## ☁️ Terraform - Cloud Infrastructure

### Providers Suportados
- AWS (EKS, RDS, ElastiCache, S3)
- GCP (GKE, Cloud SQL, Memorystore, Cloud Storage)

### Deploy AWS (exemplo)

```bash
cd terraform/environments/production

# Configure AWS credentials
export AWS_PROFILE=alta-entrega-prod

# Inicialize Terraform
terraform init

# Plan
terraform plan -out=tfplan

# Apply
terraform apply tfplan
```

### Recursos Criados
- VPC + Subnets (public/private)
- EKS Cluster (3+ worker nodes)
- RDS PostgreSQL (Multi-AZ)
- ElastiCache Redis (cluster mode)
- S3 Buckets (backups, uploads)
- Load Balancers
- Security Groups

---

## 📊 Monitoramento

### Grafana Dashboards

1. **Overview Dashboard** - Métricas gerais do sistema
2. **API Performance** - Latência, throughput, error rate
3. **Event Bus** - Redpanda lag, throughput
4. **Database** - PostgreSQL + Redis metrics
5. **Workflows** - Temporal execution stats

### Alertas (Prometheus + Alertmanager)

- API latency p99 > 500ms
- Error rate > 0.1%
- Database connections > 80%
- Redpanda lag > 10000
- Disk usage > 85%

---

## 🔐 Secrets Management

### Desenvolvimento
- `.env` files (gitignored)
- Docker secrets

### Produção
- Kubernetes Secrets (encrypted)
- AWS Secrets Manager / GCP Secret Manager
- Vault (opcional, para casos avançados)

---

## 💾 Backup & Disaster Recovery

### PostgreSQL
```bash
# Backup diário automático
./scripts/backup.sh postgres daily

# Restore
./scripts/restore.sh postgres backup-2025-02-20.sql.gz
```

### ClickHouse
```bash
./scripts/backup.sh clickhouse daily
```

### Redis
- RDB snapshots (diários)
- AOF persistence

**Recovery Targets:**
- RTO (Recovery Time Objective): < 1 hora
- RPO (Recovery Point Objective): < 15 minutos

---

## 🔄 CI/CD Pipeline

### GitOps (ArgoCD)
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: alta-entrega-core
spec:
  project: default
  source:
    repoURL: https://github.com/vinicsantana/alta-entrega-infra
    targetRevision: main
    path: kubernetes/overlays/production
  destination:
    server: https://kubernetes.default.svc
    namespace: alta-entrega-prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

---

## 📈 Scaling Strategy

### Horizontal Pod Autoscaling (HPA)
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: core-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: core-api
  minReplicas: 3
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

### Database Scaling
- **Vertical:** Increase instance size (RDS)
- **Horizontal:** Read replicas for queries
- **Partitioning:** Time-based partitions (orders, events)

---

## 🧪 Testing Infrastructure

```bash
# Teste conectividade com todos os services
./scripts/test-connectivity.sh

# Load test (k6)
k6 run tests/load/api-stress.js

# Chaos engineering (Chaos Mesh)
kubectl apply -f tests/chaos/pod-failure.yaml
```

---

## 📖 Documentação Adicional

- [Docker Compose Guide](./docs/docker-compose.md)
- [Kubernetes Setup](./docs/kubernetes.md)
- [Terraform Modules](./docs/terraform.md)
- [Monitoring & Alerting](./docs/monitoring.md)
- [Disaster Recovery Plan](./docs/disaster-recovery.md)

---

## 🤝 Contributing

Repositório proprietário. Contribuições internas apenas.

---

## 📄 License

Proprietary - All rights reserved © Alta Entrega 2025