# Domain 5: Implement an Instrumentation Strategy

**Exam Weight: 5–10%**

[← Back to Main](./README.md)

---

## 📑 Table of Contents

- [5.1 Configure Monitoring for a DevOps Environment](#51-configure-monitoring-for-a-devops-environment)
- [5.2 Analyze Metrics from Instrumentation](#52-analyze-metrics-from-instrumentation)

---

## 5.1 Configure Monitoring for a DevOps Environment

### Azure Monitor Overview

**Azure Monitor** is the **central monitoring platform** for Azure. All telemetry flows through it.

```
                     ┌──────────────────────┐
Applications ──────► │                      │
VMs/Containers ────► │    Azure Monitor     │──► Dashboards
Azure Resources ───► │   (Data Platform)    │──► Alerts
Azure DevOps ──────► │                      │──► Log Analytics
GitHub ────────────► │  Metrics | Logs |    │──► Workbooks
                     │  Traces  | Changes   │──► Power BI
                     └──────────────────────┘
```

**Azure Monitor data types:**
| Type | Storage | Examples |
|------|---------|---------|
| **Metrics** | Azure Monitor Metrics (time-series) | CPU %, response time, request count |
| **Logs** | Log Analytics workspace | Application logs, audit logs, events |
| **Traces** | Application Insights | Distributed traces across services |
| **Changes** | Change Analysis | Resource configuration changes |

---

### Application Insights

**Application Insights** is an **APM (Application Performance Monitoring)** service within Azure Monitor.

**Key capabilities:**
| Feature | Description |
|---------|-------------|
| **Live Metrics** | Real-time stream of performance data |
| **Application Map** | Visual diagram of connected services and their health |
| **Failures** | Exception tracking and failed request analysis |
| **Performance** | Response time analysis, slowest operations |
| **Users/Sessions** | Usage analytics |
| **Availability Tests** | Proactive monitoring from multiple locations |
| **Smart Detection** | AI-based anomaly detection |
| **Distributed Tracing** | End-to-end trace across microservices |

**Instrumentation approaches:**

**Option 1: SDK-based (Code-based)**
```csharp
// .NET — add Application Insights SDK
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["APPLICATIONINSIGHTS_CONNECTION_STRING"];
});
```

```python
# Python
from applicationinsights import TelemetryClient
tc = TelemetryClient('<instrumentation_key>')
tc.track_event('UserLogin', {'userId': user.id})
tc.flush()
```

**Option 2: Auto-instrumentation (Codeless)**
- Enabled via Azure App Service → **Application Insights → Enable**
- No code changes required
- Supports .NET, Java, Node.js, Python (limited)

**Custom telemetry:**
```csharp
// Track custom events
_telemetryClient.TrackEvent("OrderPlaced", 
    new Dictionary<string, string> { { "orderId", order.Id } },
    new Dictionary<string, double> { { "orderValue", order.Total } });

// Track custom metrics
_telemetryClient.TrackMetric("ActiveUsers", userCount);

// Track custom exceptions
try { ... }
catch (Exception ex)
{
    _telemetryClient.TrackException(ex, 
        new Dictionary<string, string> { { "context", "OrderService" } });
    throw;
}
```

---

### VM Insights

Monitors performance and health of **Azure VMs and VM Scale Sets**.

**Collects:**
- CPU, memory, disk I/O, network I/O
- Running processes and their performance
- TCP connections between processes

**Enable VM Insights:**
```bash
az vm extension set \
  --resource-group myRG \
  --vm-name myVM \
  --name MicrosoftMonitoringAgent \
  --publisher Microsoft.EnterpriseCloud.Monitoring \
  --settings '{"workspaceId": "<workspace-id>"}' \
  --protected-settings '{"workspaceKey": "<workspace-key>"}'
```

Or use **Azure Monitor Agent (AMA)** — the modern replacement for Log Analytics agent:
```bash
az vm extension set \
  --resource-group myRG \
  --vm-name myVM \
  --name AzureMonitorWindowsAgent \
  --publisher Microsoft.Azure.Monitor
```

---

### Container Insights

Monitors performance of **AKS (Azure Kubernetes Service)** and Azure Container Instances.

**Collects:**
- Node-level: CPU, memory, disk, network
- Pod-level: Resource requests vs limits, restarts
- Container-level: CPU, memory per container
- Kubernetes events (OOMKilled, ImagePullBackOff, etc.)

**Enable Container Insights:**
```bash
# Enable on existing AKS cluster
az aks enable-addons \
  --resource-group myRG \
  --name myAKS \
  --addons monitoring \
  --workspace-resource-id <log-analytics-workspace-id>
```

---

### Storage Insights, Network Insights

**Storage Insights:**
- Dashboard for Azure Storage accounts (Blob, Files, Queue, Table)
- Tracks: transactions, availability, performance, capacity
- Auto-enabled — no agent needed

**Network Insights:**
- Topology view of network resources
- Tracks: traffic flows, health, connectivity
- Includes: **Network Watcher**, **Traffic Analytics**, **Connection Monitor**

---

### Configure Monitoring in GitHub

**GitHub Insights (Organization level):**
- **Insights → Pulse**: Recent activity summary (PRs, issues, commits)
- **Traffic**: Repo views, clones, top referrers
- **Community**: README, contributing guide, code of conduct health
- **Dependency graph**: All dependencies (enables Dependabot)

**Creating custom charts:**
- GitHub Projects → **Insights tab** → Create charts from project data
- Track: items by status, items by assignee, burnup chart, velocity

---

### Alerts for DevOps Events

#### Azure Pipelines Alerts

```yaml
# Notify on pipeline failure via email
# Configure in: Pipeline → Notifications → Subscriptions
# OR use Service Hooks → Slack/Teams/Email

# In YAML — alert on specific condition
- script: |
    curl -X POST $TEAMS_WEBHOOK \
      -H 'Content-Type: application/json' \
      -d '{"text": "❌ Build FAILED: $(Build.DefinitionName) - $(Build.SourceBranch)"}'
  condition: failed()
  displayName: 'Notify Teams on failure'
  env:
    TEAMS_WEBHOOK: $(TeamsWebhookUrl)
```

**Azure Monitor Alert → Azure Pipelines:**
- Create an Azure Monitor action group that triggers a webhook
- Webhook calls Azure DevOps REST API to run a pipeline (auto-remediation)

#### GitHub Actions Alerts

```yaml
# Notify Slack on workflow failure
- name: Notify Slack on failure
  if: failure()
  uses: slackapi/slack-github-action@v1.26.0
  with:
    payload: |
      {
        "text": "❌ Workflow failed: ${{ github.workflow }} on ${{ github.ref }}",
        "channel": "#alerts"
      }
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

---

## 5.2 Analyze Metrics from Instrumentation

### Infrastructure Performance Indicators

| Metric | Concern Level | Description |
|--------|--------------|-------------|
| **CPU %** | > 80% sustained | High utilization — scale up or out |
| **Memory %** | > 85% sustained | Possible memory leak or undersized |
| **Disk I/O** | Latency > 20ms | Storage bottleneck |
| **Disk Space** | > 85% used | Risk of running out |
| **Network In/Out** | Bandwidth saturation | Throttling or cost concerns |
| **Network Errors** | Any non-zero | Connectivity issues |

**Alert rules in Azure Monitor:**
```bash
# Create metric alert for high CPU
az monitor metrics alert create \
  --name "High CPU Alert" \
  --resource-group myRG \
  --scopes "/subscriptions/<sub>/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM" \
  --condition "avg Percentage CPU > 80" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action "/subscriptions/<sub>/resourceGroups/myRG/providers/microsoft.insights/actionGroups/myActionGroup" \
  --description "CPU over 80% for 5 minutes"
```

---

### Application Metrics and Usage

**Application Insights — key metrics to monitor:**

| Metric | Description | Alert Threshold Example |
|--------|-------------|------------------------|
| **Server response time** | Average request duration | > 2 seconds |
| **Failed requests %** | HTTP 5xx / exceptions | > 5% |
| **Request rate** | Requests per second | Monitor for anomalies |
| **Page load time** | Client-side load duration | > 3 seconds |
| **Browser exceptions** | JavaScript errors | Monitor for spikes |
| **Availability** | % of uptime (from availability tests) | < 99.9% |

**Application Map analysis:**
- Identify slow dependencies (external APIs, databases)
- Find cascading failures
- Spot services with high error rates

---

### Distributed Tracing with Application Insights

**How distributed tracing works:**
```
Request ──► Service A ──► Service B ──► Database
              │                │
              └── Span 1       └── Span 2
                  (50ms)           (200ms)
              
Operation ID: abc-123  ← Links all spans together
```

**Correlation headers:**
- Application Insights automatically propagates `traceparent` (W3C standard) or `Request-Id` headers
- Enables end-to-end trace reconstruction across services

**Viewing traces:**
- Application Insights → **Transaction Search** → Filter by operation ID
- Application Insights → **End-to-end Transaction Details** → Waterfall view

**Sampling:**
- Telemetry can be **sampled** to reduce costs and volume
- Types: Adaptive sampling (default), Fixed-rate, Ingestion sampling
- Sampling preserves complete traces (not random events within a trace)

```csharp
// Configure adaptive sampling
builder.Services.AddApplicationInsightsTelemetry();
builder.Services.Configure<TelemetryConfiguration>(config =>
{
    var sampler = new AdaptiveSamplingTelemetryProcessor(config);
    sampler.MaxTelemetryItemsPerSecond = 20;
    sampler.InitialSamplingPercentage = 100;
});
```

---

### KQL — Kusto Query Language

**KQL** is the query language for **Log Analytics** (and Application Insights, Microsoft Sentinel, ADX).

#### KQL Basics

```kusto
// Basic query structure:
// TableName
// | operator1
// | operator2

// View all Application Insights requests in the last hour
requests
| where timestamp > ago(1h)
| order by timestamp desc
| take 100
```

**Core operators:**

| Operator | Description | Example |
|----------|-------------|---------|
| `where` | Filter rows | `where statusCode == 500` |
| `project` | Select columns | `project name, duration, success` |
| `summarize` | Aggregate | `summarize count() by bin(timestamp, 1h)` |
| `order by` | Sort | `order by duration desc` |
| `take` / `limit` | Return N rows | `take 10` |
| `extend` | Add computed column | `extend durationSec = duration / 1000` |
| `join` | Join two tables | `| join kind=inner (exceptions) on operation_Id` |
| `render` | Visualize | `| render timechart` |
| `parse` | Extract from string | `parse message with "User: " userId " logged in"` |

#### Common DevOps KQL Queries

```kusto
// --- Failed requests in last 24h ---
requests
| where timestamp > ago(24h)
| where success == false
| summarize failedCount = count() by bin(timestamp, 1h), resultCode
| render timechart

// --- Slowest operations ---
requests
| where timestamp > ago(1h)
| summarize 
    avgDuration = avg(duration),
    p95Duration = percentile(duration, 95),
    requestCount = count()
  by name
| order by p95Duration desc
| take 20

// --- Exception count by type ---
exceptions
| where timestamp > ago(24h)
| summarize count() by type
| order by count_ desc

// --- Azure Pipelines builds (via Log Analytics) ---
AzureDiagnostics
| where Category == "PipelineRuns"
| where status_s == "Failed"
| summarize count() by pipelineName_s, bin(TimeGenerated, 1d)
| render columnchart

// --- Application availability ---
availabilityResults
| where timestamp > ago(7d)
| summarize 
    availability = round(100.0 * countif(success == 1) / count(), 2)
  by location, bin(timestamp, 1d)
| render timechart

// --- Container CPU by pod ---
Perf
| where ObjectName == "K8SContainer"
| where CounterName == "cpuUsageNanoCores"
| extend PodName = extract("pod=([^,]+)", 1, InstanceName)
| summarize avgCPU = avg(CounterValue) by PodName, bin(TimeGenerated, 5m)
| render timechart
```

---

### Availability Tests

Proactively monitor your application's availability from **multiple global locations**.

**Types:**
| Type | Description |
|------|-------------|
| **URL Ping Test** | Simple HTTP GET check |
| **Standard Test** | HTTP with SSL cert validation, custom headers |
| **Multi-Step Web Test** | Sequence of HTTP requests (deprecated, use custom Track Availability) |
| **Custom Track Availability** | Call `TrackAvailability()` from Azure Function or pipeline |

**Create via Azure Portal or Bicep:**
```bicep
resource availabilityTest 'microsoft.insights/webtests@2022-06-15' = {
  name: 'ping-test-homepage'
  location: location
  tags: {
    'hidden-link:${appInsights.id}': 'Resource'
  }
  kind: 'ping'
  properties: {
    SyntheticMonitorId: 'ping-test-homepage'
    Name: 'Homepage Ping Test'
    Enabled: true
    Frequency: 300          // Every 5 minutes
    Timeout: 30
    Kind: 'ping'
    Locations: [
      { Id: 'emea-nl-ams-azr' }      // Amsterdam
      { Id: 'us-va-ash-azr' }         // East US
      { Id: 'apac-sg-sin-azr' }       // Singapore
    ]
    Configuration: {
      WebTest: '<WebTest ...>...'
    }
  }
}
```

---

## 🧠 Key Exam Tips for Instrumentation

| Scenario | Answer |
|----------|--------|
| Monitor AKS pod resource usage | **Container Insights** |
| Monitor Azure VMs CPU and memory | **VM Insights** |
| Track custom business events in application | **Application Insights** `TrackEvent()` |
| Find the slowest database queries in your app | Application Insights → **Performance → Dependencies** |
| Trace a request across 5 microservices | **Distributed Tracing** via Application Insights Application Map |
| Query failed requests in the last hour | KQL: `requests | where success == false | where timestamp > ago(1h)` |
| Get 95th percentile response time | KQL: `summarize percentile(duration, 95)` |
| Alert when error rate > 5% | Azure Monitor metric alert on `requests/failed` |
| Monitor app from 5 global locations | **Application Insights Availability Tests** |
| No code changes — enable monitoring on App Service | **Auto-instrumentation** (codeless) via App Service portal |
| Group KQL results by hour | `| summarize count() by bin(timestamp, 1h)` |
| Visualize KQL results as a time-series chart | `| render timechart` |

---

[← Security & Compliance](./04-security-and-compliance.md) | [Back to Main](./README.md) | [Resources & Cheat Sheet →](./resources.md)
