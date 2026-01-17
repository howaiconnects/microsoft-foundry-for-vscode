# Agent Zero Deployment Guide

This guide explains how to deploy Agent Zero as a meta-orchestrator with access to all project components, branches, and monitoring capabilities.

## 🏗️ Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                   Agent Zero Container                       │
│                  (Docker/Kubernetes Pod)                     │
├─────────────────────────────────────────────────────────────┤
│  Agent Zero Framework                                       │
│  ├─ Web UI (Port 8080)                                      │
│  ├─ API Server (Port 3000)                                  │
│  └─ Core Agent Engine                                       │
│                                                             │
│  Mounted Volumes:                                           │
│  ├─ /workspace → Your repository (all branches)             │
│  ├─ /configs → Agent configurations                         │
│  └─ /logs → Execution logs and metrics                      │
│                                                             │
│  Environment Variables:                                     │
│  ├─ AZURE_SUBSCRIPTION_ID                                   │
│  ├─ AZURE_RESOURCE_GROUP                                    │
│  ├─ FOUNDRY_PROJECT_NAME                                    │
│  ├─ AIRTABLE_API_KEY                                        │
│  └─ N8N_WEBHOOK_URL                                         │
└─────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
    Azure AI              Airtable              N8N Instance
    Foundry               MCP Server            Workflows
```

## 📦 Deployment Options

### Option 1: Docker Deployment (Recommended for Development/Testing)

**Advantages:**
- Quick setup and teardown
- Isolated environment
- Easy to manage
- Good for testing and development

**Prerequisites:**
```bash
# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

**Deployment Steps:**

1. **Create Deployment Directory**
```bash
mkdir -p ~/agent-zero-deployment
cd ~/agent-zero-deployment
```

2. **Create `docker-compose.yml`**
```yaml
version: '3.8'

services:
  agent-zero:
    image: agent0ai/agent-zero:latest
    container_name: foundry-meta-orchestrator
    ports:
      - "8080:8080"  # Web UI
      - "3000:3000"  # API Server
    volumes:
      # Mount your repository (read-only for safety)
      - /path/to/microsoft-foundry-for-vscode:/workspace:ro
      # Mount configurations
      - ./configs:/configs
      # Mount logs and metrics
      - ./logs:/logs
      # Mount persistent memory
      - ./agent-memory:/root/.agent-zero/memory
    environment:
      # Azure Credentials
      - AZURE_SUBSCRIPTION_ID=${AZURE_SUBSCRIPTION_ID}
      - AZURE_TENANT_ID=${AZURE_TENANT_ID}
      - AZURE_CLIENT_ID=${AZURE_CLIENT_ID}
      - AZURE_CLIENT_SECRET=${AZURE_CLIENT_SECRET}
      
      # AI Foundry Configuration
      - FOUNDRY_PROJECT_NAME=${FOUNDRY_PROJECT_NAME}
      - FOUNDRY_RESOURCE_GROUP=${FOUNDRY_RESOURCE_GROUP}
      
      # Airtable Configuration
      - AIRTABLE_API_KEY=${AIRTABLE_API_KEY}
      - AIRTABLE_BASE_ID=${AIRTABLE_BASE_ID}
      
      # N8N Configuration
      - N8N_WEBHOOK_URL=${N8N_WEBHOOK_URL}
      - N8N_API_KEY=${N8N_API_KEY}
      
      # Agent Zero Configuration
      - AGENT_ZERO_MODEL=gpt-4o
      - AGENT_ZERO_TEMPERATURE=0.3
      - AGENT_ZERO_AUTO_LEARN=true
      - AGENT_ZERO_MEMORY_PERSISTENT=true
      
      # GitHub Access (for multi-branch access)
      - GITHUB_TOKEN=${GITHUB_TOKEN}
      - GITHUB_REPO=howaiconnects/microsoft-foundry-for-vscode
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # Optional: Prometheus for metrics
  prometheus:
    image: prom/prometheus:latest
    container_name: agent-zero-metrics
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
    restart: unless-stopped

  # Optional: Grafana for dashboards
  grafana:
    image: grafana/grafana:latest
    container_name: agent-zero-dashboard
    ports:
      - "3001:3000"
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana-dashboards:/etc/grafana/provisioning/dashboards
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_INSTALL_PLUGINS=grafana-piechart-panel
    restart: unless-stopped

volumes:
  prometheus-data:
  grafana-data:
```

3. **Create `.env` File**
```bash
cat > .env << 'EOF'
# Azure Credentials
AZURE_SUBSCRIPTION_ID=your-subscription-id
AZURE_TENANT_ID=your-tenant-id
AZURE_CLIENT_ID=your-client-id
AZURE_CLIENT_SECRET=your-client-secret

# AI Foundry
FOUNDRY_PROJECT_NAME=your-foundry-project
FOUNDRY_RESOURCE_GROUP=your-resource-group

# Airtable
AIRTABLE_API_KEY=your-airtable-key
AIRTABLE_BASE_ID=your-base-id

# N8N
N8N_WEBHOOK_URL=https://your-n8n-instance.com
N8N_API_KEY=your-n8n-api-key

# GitHub (for multi-branch access)
GITHUB_TOKEN=your-github-token

# Grafana
GRAFANA_PASSWORD=secure-password
EOF

# Secure the environment file
chmod 600 .env
```

4. **Create Prometheus Configuration**
```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'agent-zero'
    static_configs:
      - targets: ['agent-zero:8080']
    metrics_path: '/metrics'
```

5. **Deploy Agent Zero**
```bash
# Start the services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f agent-zero
```

6. **Access Agent Zero**
- **Web UI**: http://localhost:8080
- **API**: http://localhost:3000
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3001 (admin/your-password)

---

### Option 2: Kubernetes Deployment (Recommended for Production)

**Advantages:**
- High availability
- Auto-scaling
- Better resource management
- Production-grade monitoring
- Multi-environment support

**Prerequisites:**
```bash
# Install kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Install helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

**Deployment Steps:**

1. **Create Namespace**
```bash
kubectl create namespace agent-zero
```

2. **Create Secrets**
```bash
# Azure credentials
kubectl create secret generic azure-credentials \
  --from-literal=subscription-id=$AZURE_SUBSCRIPTION_ID \
  --from-literal=tenant-id=$AZURE_TENANT_ID \
  --from-literal=client-id=$AZURE_CLIENT_ID \
  --from-literal=client-secret=$AZURE_CLIENT_SECRET \
  -n agent-zero

# Airtable credentials
kubectl create secret generic airtable-credentials \
  --from-literal=api-key=$AIRTABLE_API_KEY \
  --from-literal=base-id=$AIRTABLE_BASE_ID \
  -n agent-zero

# N8N credentials
kubectl create secret generic n8n-credentials \
  --from-literal=webhook-url=$N8N_WEBHOOK_URL \
  --from-literal=api-key=$N8N_API_KEY \
  -n agent-zero

# GitHub token for repository access
kubectl create secret generic github-credentials \
  --from-literal=token=$GITHUB_TOKEN \
  -n agent-zero
```

3. **Create ConfigMap**
```yaml
# agent-zero-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: agent-zero-config
  namespace: agent-zero
data:
  foundry-project: "your-foundry-project"
  resource-group: "your-resource-group"
  github-repo: "howaiconnects/microsoft-foundry-for-vscode"
  model: "gpt-4o"
  temperature: "0.3"
  auto-learn: "true"
```

```bash
kubectl apply -f agent-zero-config.yaml
```

4. **Create Persistent Volumes**
```yaml
# agent-zero-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: agent-zero-memory
  namespace: agent-zero
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
  storageClassName: standard

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: agent-zero-logs
  namespace: agent-zero
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 20Gi
  storageClassName: standard
```

```bash
kubectl apply -f agent-zero-pvc.yaml
```

5. **Create Deployment**
```yaml
# agent-zero-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: agent-zero-meta-orchestrator
  namespace: agent-zero
spec:
  replicas: 1  # Single instance for consistent state
  selector:
    matchLabels:
      app: agent-zero
  template:
    metadata:
      labels:
        app: agent-zero
    spec:
      serviceAccountName: agent-zero-sa
      containers:
      - name: agent-zero
        image: agent0ai/agent-zero:latest
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          name: web-ui
        - containerPort: 3000
          name: api
        env:
        # Azure credentials from secret
        - name: AZURE_SUBSCRIPTION_ID
          valueFrom:
            secretKeyRef:
              name: azure-credentials
              key: subscription-id
        - name: AZURE_TENANT_ID
          valueFrom:
            secretKeyRef:
              name: azure-credentials
              key: tenant-id
        - name: AZURE_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: azure-credentials
              key: client-id
        - name: AZURE_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: azure-credentials
              key: client-secret
        
        # Foundry configuration from ConfigMap
        - name: FOUNDRY_PROJECT_NAME
          valueFrom:
            configMapKeyRef:
              name: agent-zero-config
              key: foundry-project
        - name: FOUNDRY_RESOURCE_GROUP
          valueFrom:
            configMapKeyRef:
              name: agent-zero-config
              key: resource-group
        
        # Airtable credentials
        - name: AIRTABLE_API_KEY
          valueFrom:
            secretKeyRef:
              name: airtable-credentials
              key: api-key
        
        # N8N credentials
        - name: N8N_WEBHOOK_URL
          valueFrom:
            secretKeyRef:
              name: n8n-credentials
              key: webhook-url
        
        # GitHub access
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: github-credentials
              key: token
        - name: GITHUB_REPO
          valueFrom:
            configMapKeyRef:
              name: agent-zero-config
              key: github-repo
        
        # Agent Zero settings
        - name: AGENT_ZERO_MODEL
          valueFrom:
            configMapKeyRef:
              name: agent-zero-config
              key: model
        - name: AGENT_ZERO_TEMPERATURE
          valueFrom:
            configMapKeyRef:
              name: agent-zero-config
              key: temperature
        
        volumeMounts:
        - name: memory
          mountPath: /root/.agent-zero/memory
        - name: logs
          mountPath: /logs
        
        resources:
          requests:
            cpu: "1"
            memory: "2Gi"
          limits:
            cpu: "4"
            memory: "8Gi"
        
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
      
      volumes:
      - name: memory
        persistentVolumeClaim:
          claimName: agent-zero-memory
      - name: logs
        persistentVolumeClaim:
          claimName: agent-zero-logs
```

```bash
kubectl apply -f agent-zero-deployment.yaml
```

6. **Create Service**
```yaml
# agent-zero-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: agent-zero-service
  namespace: agent-zero
spec:
  type: LoadBalancer
  selector:
    app: agent-zero
  ports:
  - name: web-ui
    port: 80
    targetPort: 8080
  - name: api
    port: 3000
    targetPort: 3000
```

```bash
kubectl apply -f agent-zero-service.yaml
```

7. **Create Ingress (Optional)**
```yaml
# agent-zero-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: agent-zero-ingress
  namespace: agent-zero
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - agent-zero.yourdomain.com
    secretName: agent-zero-tls
  rules:
  - host: agent-zero.yourdomain.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: agent-zero-service
            port:
              number: 80
```

```bash
kubectl apply -f agent-zero-ingress.yaml
```

---

## 🔐 Multi-Branch Access Configuration

Agent Zero needs access to all branches in your repository. Here's how to configure it:

### Method 1: Git Clone with Full History (Recommended)

```python
# In agent-zero-config.py, add this initialization code:

import git
import os

def initialize_repository_access():
    """
    Clone repository with all branches for Agent Zero access
    """
    repo_path = "/workspace"
    
    if not os.path.exists(repo_path):
        # Clone with all branches
        repo = git.Repo.clone_from(
            f"https://{os.getenv('GITHUB_TOKEN')}@github.com/howaiconnects/microsoft-foundry-for-vscode.git",
            repo_path,
            mirror=True
        )
    else:
        repo = git.Repo(repo_path)
        
        # Fetch all branches
        repo.remotes.origin.fetch()
    
    # Get all branches
    branches = [ref.name for ref in repo.references if ref.name.startswith('refs/heads/')]
    
    print(f"Available branches: {branches}")
    return repo, branches

# Register as Agent Zero tool
@meta_orchestrator.tool
class RepositoryAccessTool(Tool):
    """Tool for accessing repository branches and configurations"""
    
    def __init__(self):
        self.repo, self.branches = initialize_repository_access()
    
    def list_branches(self):
        """List all available branches"""
        return self.branches
    
    def checkout_branch(self, branch_name):
        """Switch to specific branch"""
        self.repo.git.checkout(branch_name)
        return f"Switched to branch: {branch_name}"
    
    def get_architecture_configs(self, branch_name="main"):
        """Get all architecture configurations from a branch"""
        self.checkout_branch(branch_name)
        
        configs_path = "tutorials/agentic-organizational-structures"
        configs = {}
        
        for level in ["level-1", "level-2", "level-3", "level-4", "level-5", "level-6"]:
            level_path = os.path.join(configs_path, level)
            if os.path.exists(level_path):
                configs[level] = self._load_level_configs(level_path)
        
        return configs
    
    def compare_branch_architectures(self, branch1, branch2):
        """Compare architectures between branches"""
        configs1 = self.get_architecture_configs(branch1)
        configs2 = self.get_architecture_configs(branch2)
        
        # Generate comparison
        return self._generate_comparison(configs1, configs2)
```

### Method 2: GitHub API Access

```python
from github import Github

@meta_orchestrator.tool
class GitHubAccessTool(Tool):
    """Tool for accessing GitHub repository via API"""
    
    def __init__(self):
        self.github = Github(os.getenv('GITHUB_TOKEN'))
        self.repo = self.github.get_repo("howaiconnects/microsoft-foundry-for-vscode")
    
    def list_branches(self):
        """List all branches in repository"""
        return [branch.name for branch in self.repo.get_branches()]
    
    def get_file_content(self, file_path, branch="main"):
        """Get file content from specific branch"""
        content = self.repo.get_contents(file_path, ref=branch)
        return content.decoded_content.decode('utf-8')
    
    def list_architecture_files(self, branch="main"):
        """List all architecture configuration files"""
        contents = self.repo.get_contents(
            "tutorials/agentic-organizational-structures",
            ref=branch
        )
        
        files = []
        while contents:
            content = contents.pop(0)
            if content.type == "dir":
                contents.extend(self.repo.get_contents(content.path, ref=branch))
            elif content.name.endswith('.yaml'):
                files.append(content.path)
        
        return files
```

---

## 📊 Progress Monitoring

Agent Zero provides multiple ways to monitor progress:

### 1. Web UI Dashboard

Access at `http://localhost:8080` (Docker) or your Kubernetes ingress URL.

**Features:**
- Real-time task execution view
- Architecture deployment status
- Performance metrics graphs
- Comparison results
- Logs and error tracking

**Dashboard Sections:**

```
┌─────────────────────────────────────────────────────────────┐
│  Agent Zero Meta-Orchestrator Dashboard                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Active Deployments:                                        │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ Level-3-TwoTier      │ Running  │ 45s latency    │  │
│  │ Level-4-ProjectMgr   │ Running  │ 62s latency    │  │
│  │ Level-5-TeamHierarch │ Starting │ -              │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Current Tasks:                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ✓ Deploy Level-3 architecture                        │  │
│  │ ✓ Configure Airtable MCP server                      │  │
│  │ → Run benchmarking tests (45% complete)              │  │
│  │ ○ Generate comparison report                         │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  Performance Metrics:                                       │
│  ┌───────────────┬───────────────┬───────────────┐       │
│  │   Latency     │  Success Rate │  Cost/Run     │       │
│  │     [Graph]   │     [Graph]   │   [Graph]     │       │
│  └───────────────┴───────────────┴───────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

### 2. REST API Monitoring

```python
import requests

API_BASE = "http://localhost:3000"

# Get current status
response = requests.get(f"{API_BASE}/api/status")
status = response.json()

print(f"Status: {status['state']}")
print(f"Active tasks: {status['active_tasks']}")
print(f"Deployments: {status['deployments']}")

# Get deployment details
deployment_id = "prod-customer-onboarding-001"
response = requests.get(f"{API_BASE}/api/deployments/{deployment_id}")
deployment = response.json()

print(f"Architecture: {deployment['architecture']}")
print(f"Status: {deployment['status']}")
print(f"Metrics: {deployment['metrics']}")

# Get comparison results
response = requests.get(f"{API_BASE}/api/comparisons/latest")
comparison = response.json()

print(f"Compared: {comparison['architectures']}")
print(f"Winner: {comparison['recommended']}")
print(f"Reason: {comparison['reasoning']}")

# Subscribe to real-time updates (WebSocket)
import websocket

ws = websocket.WebSocket()
ws.connect("ws://localhost:3000/ws")

while True:
    message = ws.recv()
    event = json.loads(message)
    
    if event['type'] == 'deployment_update':
        print(f"Deployment {event['deployment_id']}: {event['status']}")
    elif event['type'] == 'task_progress':
        print(f"Task {event['task_id']}: {event['progress']}%")
    elif event['type'] == 'comparison_complete':
        print(f"Comparison complete: {event['results']}")
```

### 3. Prometheus Metrics

Query Prometheus at `http://localhost:9090`:

```promql
# Average deployment latency
avg(agent_zero_deployment_latency_seconds) by (architecture)

# Success rate by architecture
rate(agent_zero_deployment_success_total[5m]) / 
rate(agent_zero_deployment_total[5m])

# Current active deployments
agent_zero_active_deployments

# Resource utilization
agent_zero_cpu_usage_percent
agent_zero_memory_usage_bytes

# Task queue depth
agent_zero_task_queue_depth
```

### 4. Grafana Dashboards

Access at `http://localhost:3001`:

**Pre-configured Dashboards:**

1. **Overview Dashboard**
   - Active deployments count
   - Success rate trends
   - Latency distribution
   - Cost per operation

2. **Architecture Comparison Dashboard**
   - Side-by-side metrics comparison
   - Performance trends over time
   - Cost analysis
   - Recommendation confidence scores

3. **System Health Dashboard**
   - Agent Zero resource usage
   - Task execution times
   - Error rates and types
   - Queue depths

### 5. Log Monitoring

```bash
# Docker logs
docker-compose logs -f agent-zero

# Kubernetes logs
kubectl logs -f deployment/agent-zero-meta-orchestrator -n agent-zero

# Tail specific log files
tail -f ./logs/deployments.log
tail -f ./logs/comparisons.log
tail -f ./logs/optimizations.log
```

**Log Structure:**
```
[2026-01-17 02:00:00] [INFO] Starting architecture comparison
[2026-01-17 02:00:05] [INFO] Deployed level-3-two-tier (deployment-001)
[2026-01-17 02:00:10] [INFO] Deployed level-4-project-manager (deployment-002)
[2026-01-17 02:05:00] [INFO] Benchmark progress: 45% (450/1000 tests)
[2026-01-17 02:10:00] [INFO] Benchmark complete: level-3 avg=45s, level-4 avg=62s
[2026-01-17 02:10:05] [INFO] Generating comparison report
[2026-01-17 02:10:10] [INFO] Comparison complete: Recommended level-4
```

### 6. Slack/Email Notifications

Configure notifications in Agent Zero:

```python
# In agent-zero-config.py
notification_config = {
    'slack': {
        'enabled': True,
        'webhook_url': os.getenv('SLACK_WEBHOOK_URL'),
        'events': ['deployment_complete', 'comparison_complete', 'error']
    },
    'email': {
        'enabled': True,
        'smtp_server': os.getenv('SMTP_SERVER'),
        'recipients': ['team@example.com'],
        'events': ['deployment_failed', 'optimization_complete']
    }
}

meta_orchestrator.configure_notifications(notification_config)
```

---

## 🔄 Complete Workflow Example

Here's a complete example showing Agent Zero deploying architectures with full monitoring:

```python
# Initialize Agent Zero
orchestrator = AIFoundryMetaOrchestrator(
    subscription_id=os.getenv('AZURE_SUBSCRIPTION_ID'),
    resource_group=os.getenv('FOUNDRY_RESOURCE_GROUP'),
    project_name=os.getenv('FOUNDRY_PROJECT_NAME')
)

# Compare architectures with progress monitoring
async def run_comparison_with_monitoring():
    # Start comparison
    comparison_task = orchestrator.compare_architectures(
        architectures=["level-3-two-tier", "level-4-project-manager", "level-5-team-hierarchy"],
        use_case="customer_onboarding",
        test_duration="24h"
    )
    
    # Monitor progress
    while not comparison_task.is_complete():
        status = await comparison_task.get_status()
        
        print(f"Progress: {status['progress']}%")
        print(f"Completed tests: {status['completed_tests']}/{status['total_tests']}")
        print(f"Current metrics:")
        for arch, metrics in status['current_metrics'].items():
            print(f"  {arch}: {metrics['avg_latency']}s latency, {metrics['success_rate']}% success")
        
        await asyncio.sleep(30)  # Check every 30 seconds
    
    # Get final results
    results = await comparison_task.get_results()
    
    print("\n=== Comparison Complete ===")
    print(f"Recommended: {results['recommended']}")
    print(f"Reason: {results['reasoning']}")
    print("\nFull Results:")
    print(json.dumps(results, indent=2))
    
    return results

# Run the comparison
results = asyncio.run(run_comparison_with_monitoring())
```

---

## 🔧 Troubleshooting

### Issue: Agent Zero can't access repository branches

**Solution:**
```bash
# Verify GitHub token has correct permissions
curl -H "Authorization: token $GITHUB_TOKEN" \
  https://api.github.com/repos/howaiconnects/microsoft-foundry-for-vscode/branches

# Mount repository with correct permissions
docker run -v /path/to/repo:/workspace:ro ...
```

### Issue: Can't access Web UI

**Solution:**
```bash
# Check if container is running
docker ps | grep agent-zero

# Check logs
docker logs agent-zero

# Verify port mapping
netstat -tulpn | grep 8080
```

### Issue: Deployments failing

**Solution:**
```bash
# Check Azure credentials
az account show

# Verify AI Foundry access
az ml workspace show --name $FOUNDRY_PROJECT_NAME \
  --resource-group $FOUNDRY_RESOURCE_GROUP

# Check logs for specific error
docker logs agent-zero 2>&1 | grep ERROR
```

---

## 📚 Next Steps

1. **Deploy Agent Zero** using Docker or Kubernetes
2. **Configure access** to all project components
3. **Start with a comparison** of Level 3, 4, and 5
4. **Monitor progress** via Web UI and APIs
5. **Review results** and deploy optimal architecture
6. **Enable self-optimization** for continuous improvement

For detailed implementation examples, see [`agent-zero-config.py`](./agent-zero-config.py).
