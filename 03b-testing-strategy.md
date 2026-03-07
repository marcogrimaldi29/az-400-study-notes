# 3b: Design and Implement a Testing Strategy for Pipelines

[← Back to Domain 3](./03-build-and-release-pipelines.md)

---

## Quality and Release Gates

### What Are Quality Gates?

Quality gates are **automated checkpoints** in a pipeline that enforce minimum standards before code progresses to the next stage.

```
Build ──► [Quality Gate] ──► Staging ──► [Release Gate] ──► Production
               │                               │
           Must Pass:                      Must Pass:
           - Unit tests                    - Integration tests
           - Code coverage ≥ 80%           - Load test baseline
           - No critical vulnerabilities   - Manual approval
           - Static analysis               - Monitoring health
```

### Azure Pipelines — Gates and Checks

**Pre-deployment gates (Classic pipelines):**
| Gate Type | Description |
|-----------|-------------|
| **Azure Monitor Alerts** | Check no active alerts in a window |
| **Invoke REST API** | Call custom API endpoint, validate response |
| **Query Azure Monitor** | Check specific metric thresholds |
| **Query Work Items** | Ensure no open critical bugs |
| **Security and compliance assessment** | Microsoft Defender for Cloud |

**YAML-based checks (Environments):**
```yaml
# Configure in Environments UI, then reference:
- stage: DeployProd
  jobs:
    - deployment: Deploy
      environment: 'production'   # ← Gates/approvals configured on this environment
      strategy:
        runOnce:
          deploy:
            steps:
              - script: echo "Deploying to production"
```

**In Environment settings, configure:**
- Required reviewers (manual approval)
- Branch control (only deploy from `main`)
- Business hours check
- Invoke REST API gate
- Query Azure Monitor alerts

---

### GitHub Actions — Environments and Protection Rules

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://myapp.example.com
    steps:
      - name: Deploy
        run: ./deploy.sh
```

**Environment protection rules (configured in GitHub UI):**
- **Required reviewers**: Named users/teams must approve
- **Wait timer**: Delay before deployment (e.g., 10 minutes)
- **Deployment branches**: Only from `main` or `release/*`

---

## Comprehensive Testing Strategy

### Testing Pyramid

```
                    /\
                   /  \
                  / UI \          ← Few, slow, expensive
                 /______\
                /        \
               / Integration\    ← Some
              /______________\
             /                \
            /    Unit Tests    \  ← Many, fast, cheap
           /____________________\
```

| Test Type | Scope | Speed | Cost | Run In |
|-----------|-------|-------|------|--------|
| **Unit** | Single function/class | Very fast (<1s) | Low | Every build |
| **Integration** | Multiple components | Moderate | Medium | Every build/PR |
| **End-to-End (E2E)** | Full user workflow | Slow | High | Nightly/pre-release |
| **Load/Performance** | System under stress | Very slow | High | Pre-release |
| **Contract** | API consumer/provider | Fast | Low | Every build |
| **Security/SAST** | Source code | Moderate | Low | Every build |
| **DAST** | Running application | Slow | Medium | Staging |

---

### Unit Tests

**Characteristics:**
- Test a **single unit** (function, method, class) in isolation
- Use **mocks/stubs** for dependencies
- Must be **fast** (milliseconds per test)
- Must be **deterministic** (same input → same output, always)
- **No network, file system, or database access**

**Example (C# / xUnit):**
```csharp
[Fact]
public void CalculateTotal_WithDiscount_ReturnsCorrectAmount()
{
    // Arrange
    var calculator = new OrderCalculator();
    var order = new Order { Items = new[] { new Item { Price = 100m } } };
    
    // Act
    var total = calculator.CalculateTotal(order, discountPercent: 10);
    
    // Assert
    Assert.Equal(90m, total);
}
```

**Pipeline (Azure Pipelines):**
```yaml
- task: DotNetCoreCLI@2
  displayName: 'Run Unit Tests'
  inputs:
    command: 'test'
    projects: '**/*Tests/*.csproj'
    arguments: '--configuration Release --collect:"XPlat Code Coverage"'
    publishTestResults: true
```

---

### Integration Tests

**Characteristics:**
- Test **multiple components working together**
- May use test databases, in-memory services, or Docker containers
- Slower than unit tests
- Reveal interface/contract issues between components

**Service containers in Azure Pipelines:**
```yaml
jobs:
  - job: IntegrationTests
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_DB: testdb
          POSTGRES_USER: testuser
          POSTGRES_PASSWORD: testpass
        ports:
          - 5432:5432
    steps:
      - script: dotnet test --filter Category=Integration
        env:
          DB_CONNECTION: "Host=localhost;Database=testdb;Username=testuser;Password=testpass"
```

**GitHub Actions service containers:**
```yaml
services:
  redis:
    image: redis:7
    ports:
      - 6379:6379
    options: >-
      --health-cmd "redis-cli ping"
      --health-interval 10s
      --health-timeout 5s
      --health-retries 5
```

---

### Load Tests

**Azure Load Testing:**
- Cloud-based, managed load testing service
- Based on **Apache JMeter** scripts or URL-based tests
- Can be integrated into CI/CD pipelines

**Pipeline integration:**
```yaml
- task: AzureLoadTest@1
  inputs:
    azureSubscription: 'MyServiceConnection'
    loadTestConfigFile: 'load-test.yaml'
    resourceGroup: 'my-rg'
    loadTestResource: 'my-load-test'
    failCriteria: |
      avg(response_time_ms) > 2000
      percentage(error) > 5
```

**Load test types:**
| Type | Pattern | Goal |
|------|---------|------|
| **Load test** | Constant load | Verify behaviour under expected load |
| **Stress test** | Increasing load until failure | Find breaking point |
| **Spike test** | Sudden burst | Test response to unexpected traffic |
| **Endurance/Soak test** | Sustained load over time | Detect memory leaks |

---

## Implementing Tests in a Pipeline

### Configuring Test Tasks

**Azure Pipelines test tasks:**

| Task | Package Type |
|------|-------------|
| `DotNetCoreCLI@2` | .NET (xUnit, NUnit, MSTest) |
| `VSTest@2` | Visual Studio test runner |
| `Maven@3` | Maven (JUnit) |
| `Gradle@3` | Gradle (JUnit) |
| `Npm@1` | Node.js (Jest, Mocha) |
| `PublishTestResults@2` | Publish any test results (JUnit XML format) |

**Publishing test results (generic):**
```yaml
- task: PublishTestResults@2
  condition: succeededOrFailed()   # ← Always publish, even if tests fail
  inputs:
    testResultsFormat: 'JUnit'     # JUnit, NUnit, VSTest, XUnit, CTest
    testResultsFiles: '**/test-results/*.xml'
    mergeTestResults: true
    testRunTitle: 'Unit Tests'
```

### Configuring Test Agents / Runners

**Azure DevOps Agent pools:**
```yaml
pool:
  name: 'Default'          # Self-hosted pool
  # OR
  vmImage: 'ubuntu-latest' # Microsoft-hosted

# Matrix strategy for multi-platform testing:
strategy:
  matrix:
    linux:
      pool.vmImage: 'ubuntu-latest'
    windows:
      pool.vmImage: 'windows-latest'
    mac:
      pool.vmImage: 'macOS-latest'
```

**Parallel test execution:**
```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    arguments: '--parallel'   # Run test projects in parallel
```

**Test impact analysis (Azure DevOps):**
- Runs only tests affected by code changes
- Reduces test execution time significantly
- Enable in VSTest task: `runOnlyImpactedTests: true`

---

## Code Coverage Analysis

### Collecting Coverage

**Azure Pipelines (.NET):**
```yaml
- task: DotNetCoreCLI@2
  inputs:
    command: 'test'
    arguments: '--collect:"XPlat Code Coverage" --results-directory $(Agent.TempDirectory)'

- task: PublishCodeCoverageResults@2
  inputs:
    codeCoverageTool: 'Cobertura'
    summaryFileLocation: '$(Agent.TempDirectory)/**/coverage.cobertura.xml'
```

**GitHub Actions (.NET with Coverlet):**
```yaml
- name: Test with coverage
  run: dotnet test --collect:"XPlat Code Coverage"

- name: Upload coverage to Codecov
  uses: codecov/codecov-action@v4
  with:
    token: ${{ secrets.CODECOV_TOKEN }}
    files: '**/coverage.cobertura.xml'
    fail_ci_if_error: true
```

### Enforcing Coverage Thresholds

**.NET — enforce in test command:**
```bash
dotnet test /p:CollectCoverage=true \
            /p:CoverletOutputFormat=cobertura \
            /p:Threshold=80 \
            /p:ThresholdType=line \
            /p:ThresholdStat=minimum
```

**SonarQube/SonarCloud quality gate:**
- Configure coverage threshold in Quality Gate settings
- Add `SonarQubePrepare`, `SonarQubeAnalyze`, `SonarQubePublish` tasks
- Pipeline fails if quality gate not met

**Coverage types:**
| Type | Measures |
|------|---------|
| **Line coverage** | % of code lines executed |
| **Branch coverage** | % of decision branches taken (if/else, switch) |
| **Function coverage** | % of functions called |
| **Statement coverage** | % of statements executed |

> ⭐ **Branch coverage** is more meaningful than line coverage for complex logic.

---

## 🧠 Key Exam Tips for Testing Strategy

| Scenario | Answer |
|----------|--------|
| Block deployment if code coverage drops below 80% | Configure coverage threshold + quality gate in SonarQube or `dotnet test /p:Threshold=80` |
| Run tests on multiple OS versions in one pipeline | Matrix strategy |
| Only run tests affected by the code change | Test Impact Analysis (Azure DevOps VSTest task) |
| Verify production is healthy before completing deployment | Post-deployment gate (query Azure Monitor alerts) |
| Test a database integration in pipeline without external DB | Service containers (Docker) in pipeline job |
| Detect performance regression | Azure Load Testing integrated into pipeline with `failCriteria` |
| Prevent merge if pipeline tests fail | Branch policy: Build validation |
| Store test results for reporting | `PublishTestResults@2` with `condition: succeededOrFailed()` |

---

[← Package Management](./03a-package-management.md) | [Next: Pipelines →](./03c-pipelines.md)
