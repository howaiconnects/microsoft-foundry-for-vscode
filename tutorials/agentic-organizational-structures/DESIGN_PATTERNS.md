# Agentic Design Patterns & Architectures

This document provides a comprehensive reference of design patterns, architectural stacks, components, and tools for building agentic systems with Microsoft AI Foundry.

## 📐 Architectural Patterns

### 1. Single Agent Pattern
**When to Use**: Simple, focused tasks with limited scope

**Structure**:
```
User → Agent → Tool(s) → Response
```

**Characteristics**:
- One agent handles entire workflow
- Minimal coordination overhead
- Easy to deploy and maintain
- Limited scalability

**Best For**:
- Chatbots and assistants
- Simple automation tasks
- Proof of concepts
- Narrow domain expertise

**Example**: Customer FAQ bot, code review assistant, document summarizer

---

### 2. Orchestrator-Worker Pattern
**When to Use**: Tasks requiring multiple specialized capabilities

**Structure**:
```
        Orchestrator
             │
    ┌────────┼────────┐
    ▼        ▼        ▼
 Worker1  Worker2  Worker3
```

**Characteristics**:
- Central coordinator
- Specialized workers
- Clear task delegation
- Moderate complexity

**Best For**:
- Multi-step processes
- Domain specialization
- Parallel task execution
- 5-15 agents total

**Example**: Data pipeline (fetch → validate → analyze → report)

---

### 3. Hierarchical Team Pattern
**When to Use**: Large-scale systems requiring organization

**Structure**:
```
    Executive Orchestrator
            │
    ┌───────┼───────┐
    ▼       ▼       ▼
  Team   Team    Team
  Lead   Lead    Lead
    │      │       │
 ┌──┼──┐ ┌┼┐   ┌──┼──┐
 ▼  ▼  ▼ ▼ ▼   ▼  ▼  ▼
Sub-Agents per team
```

**Characteristics**:
- Multi-level hierarchy
- Team-based organization
- Clear reporting lines
- High scalability

**Best For**:
- Enterprise systems
- 20+ agents
- Department-based org
- Complex coordination

**Example**: Enterprise CRM with Data, Analytics, Operations, and Customer Success teams

---

### 4. Event-Driven Pattern
**When to Use**: Reactive systems responding to external events

**Structure**:
```
Events → Event Bus → Agents → Actions
              ↕
           N8N Workflows
```

**Characteristics**:
- Reactive rather than proactive
- Decoupled components
- Asynchronous processing
- High throughput

**Best For**:
- Real-time monitoring
- Alert systems
- Automated responses
- Integration hubs

**Example**: SLA monitoring, inventory alerts, support ticket routing

---

### 5. Pipeline Pattern
**When to Use**: Sequential data processing with transformations

**Structure**:
```
Input → Stage1 → Stage2 → Stage3 → Output
        Agent1   Agent2   Agent3
```

**Characteristics**:
- Linear flow
- Each stage transforms data
- Quality gates between stages
- Predictable execution

**Best For**:
- ETL processes
- Content pipelines
- Data enrichment
- Report generation

**Example**: Data ingestion → cleaning → analysis → visualization → report

---

### 6. Mesh Pattern
**When to Use**: Peer-to-peer agent collaboration

**Structure**:
```
  Agent1 ←→ Agent2
    ↕          ↕
  Agent3 ←→ Agent4
```

**Characteristics**:
- No central coordinator
- Peer-to-peer communication
- Emergent behavior
- Complex coordination

**Best For**:
- Distributed decision-making
- Consensus systems
- Multi-agent negotiation
- Research and exploration

**Example**: Multi-agent debate, collaborative problem-solving

---

## 🧰 Component Types

### Orchestrators
**Purpose**: Coordinate multiple agents

**Types**:
1. **Simple Router**: Routes to single agent based on rules
2. **Workflow Manager**: Manages multi-step processes
3. **Resource Manager**: Allocates and tracks resources
4. **Strategic Planner**: High-level planning and coordination

**Key Features**:
- Task decomposition
- Agent selection logic
- Dependency management
- Result aggregation

---

### Worker Agents
**Purpose**: Execute specialized tasks

**Types**:
1. **Data Workers**: Query, validate, transform data
2. **Analysis Workers**: Analyze, calculate, predict
3. **Generation Workers**: Create content, reports, code
4. **Integration Workers**: Interface with external systems
5. **Validation Workers**: Check quality, compliance

**Key Features**:
- Narrow specialization
- Idempotent operations
- Clear input/output contracts
- Error reporting

---

### Tool Integrations
**Purpose**: Extend agent capabilities

**Categories**:

**1. Data Access Tools**
- **MCP Servers**: Airtable, databases, file systems
- **File Search**: Vector stores, document search
- **Code Interpreter**: Execute Python for analysis

**2. Web & API Tools**
- **OpenAPI**: REST API access
- **Bing Grounding**: Web search
- **Custom Webhooks**: External service calls

**3. Communication Tools**
- **Email**: SendGrid, Office 365
- **Chat**: Slack, Teams
- **Notifications**: Push, SMS

**4. Workflow Tools**
- **N8N**: Workflow automation
- **Schedulers**: Cron, time-based triggers
- **Event Systems**: Webhooks, message queues

---

## 🔧 Integration Stacks

### Stack 1: Basic Agent + Airtable
**Complexity**: ⭐ Simple

```yaml
Components:
- Single AI agent
- Airtable MCP server
- Direct user interaction

Use Cases:
- Personal productivity
- Simple data management
- Small team coordination
```

---

### Stack 2: Multi-Agent + Airtable + File Search
**Complexity**: ⭐⭐ Moderate

```yaml
Components:
- Orchestrator agent
- 3-5 worker agents
- Airtable MCP server
- File Search for documentation
- Vector store for memory

Use Cases:
- Department automation
- Knowledge management
- Medium-scale workflows
```

---

### Stack 3: Hierarchical + Airtable + N8N
**Complexity**: ⭐⭐⭐⭐ Advanced

```yaml
Components:
- Executive orchestrator
- 3-4 team leads
- 15-25 worker agents
- Airtable (data store)
- N8N (workflow automation)
- Multiple MCP servers
- Event bus
- Monitoring stack

Use Cases:
- Enterprise operations
- Multi-department systems
- Complex business processes
```

---

### Stack 4: Full Production System
**Complexity**: ⭐⭐⭐⭐⭐ Expert

```yaml
Components:
- Multi-tier agent hierarchy
- N8N workflow platform
- Airtable + additional databases
- API gateway
- Load balancer
- Message queue
- Caching layer (Redis)
- Monitoring (APM, logs, metrics)
- Analytics platform
- Backup and DR

Use Cases:
- Mission-critical systems
- High-volume processing
- Multi-tenant platforms
- Global deployments
```

---

## 🎨 Design Principles

### 1. Separation of Concerns
**Principle**: Each agent has one clear responsibility

**Good**:
```yaml
- Data Fetcher Agent: Only fetches data
- Validator Agent: Only validates
- Analyzer Agent: Only analyzes
```

**Bad**:
```yaml
- Super Agent: Fetches, validates, analyzes, reports (too much)
```

---

### 2. Loose Coupling
**Principle**: Agents don't depend on implementation details of others

**Good**:
```yaml
Orchestrator → [Standard Interface] → Worker
- Clear input/output contracts
- Workers are interchangeable
```

**Bad**:
```yaml
Agent A → [Specific implementation details] → Agent B
- Tight coupling
- Hard to replace or update
```

---

### 3. Fail Fast, Fail Gracefully
**Principle**: Detect errors early, handle them gracefully

**Implementation**:
```yaml
- Validate inputs immediately
- Return clear error messages
- Provide partial results when possible
- Log errors for debugging
- Implement retry logic
- Have fallback strategies
```

---

### 4. Idempotency
**Principle**: Repeated operations produce same result

**Implementation**:
```yaml
- Use unique IDs for operations
- Check if operation already completed
- Design safe-to-retry operations
- Avoid side effects in queries
```

---

### 5. Observable Systems
**Principle**: Monitor, log, and measure everything

**Implementation**:
```yaml
- Log all agent invocations
- Track success/failure rates
- Measure response times
- Monitor resource usage
- Capture business metrics
- Set up alerting
```

---

## 📊 Decision Trees

### When to Add an Orchestrator?

```
Do you have >5 specialized agents?
├─ YES → Need orchestrator
└─ NO → Are workflows complex?
    ├─ YES → Consider orchestrator
    └─ NO → Single agent sufficient
```

### When to Use N8N vs Direct Agent?

```
Does task require multi-system coordination?
├─ YES → Use N8N
└─ NO → Is it scheduled/recurring?
    ├─ YES → Use N8N
    └─ NO → Does it need complex error recovery?
        ├─ YES → Use N8N
        └─ NO → Direct agent call
```

### When to Create a New Team?

```
Is team lead managing >10 agents?
├─ YES → Split team
└─ NO → Are there distinct functional areas?
    ├─ YES → Consider new team
    └─ NO → Is there a bottleneck?
        ├─ YES → May need new team
        └─ NO → Current structure OK
```

---

## 🔍 Common Anti-Patterns to Avoid

### ❌ God Agent
**Problem**: One agent trying to do everything

**Solution**: Split into specialized agents with orchestrator

---

### ❌ Circular Dependencies
**Problem**: Agent A calls Agent B calls Agent A

**Solution**: Redesign with clear hierarchy or shared data store

---

### ❌ Chatty Agents
**Problem**: Too many small interactions between agents

**Solution**: Batch operations, pass more context upfront

---

### ❌ No Error Handling
**Problem**: Failures cascade through system

**Solution**: Implement error handling at every level

---

### ❌ Hardcoded Logic
**Problem**: Business rules embedded in agent instructions

**Solution**: Externalize configuration, use data-driven approach

---

### ❌ No Monitoring
**Problem**: Can't diagnose issues or measure performance

**Solution**: Implement comprehensive logging and monitoring

---

## 📚 Quick Reference: Tool Selection

| Need | Tool | When to Use |
|------|------|-------------|
| Database queries | MCP (Airtable) | Structured data, CRUD operations |
| Document search | File Search | Large document corpus, semantic search |
| Data analysis | Code Interpreter | Calculations, charts, data transformation |
| Web search | Bing Grounding | Current information, web data |
| API calls | OpenAPI | REST APIs, external services |
| Multi-system | N8N | Orchestrating multiple tools/APIs |
| Scheduling | N8N Cron | Recurring tasks, time-based triggers |
| Events | N8N Webhooks | Real-time reactions, external events |

---

## 🎯 Pattern Selection Matrix

| Scenario | Pattern | Complexity | Agents | Tools |
|----------|---------|------------|--------|-------|
| Simple FAQ bot | Single Agent | ⭐ | 1 | None |
| Data pipeline | Pipeline | ⭐⭐ | 3-5 | MCP, Code Interpreter |
| Customer support | Orchestrator-Worker | ⭐⭐⭐ | 5-10 | MCP, OpenAPI, Bing |
| Enterprise CRM | Hierarchical | ⭐⭐⭐⭐ | 20-50 | All tools + N8N |
| Alert system | Event-Driven | ⭐⭐⭐ | 5-15 | N8N, MCP, Webhooks |

---

## 🔗 Related Resources

- [AI Foundry Agent Schema](https://aka.ms/ai-foundry-vsc/agent/1.0.0)
- [Sample Agents in Repository](../../samples/agents/)
- [Level 1: Single Agent](./level-1-single-agent/)
- [Level 6: Full Integration](./level-6-n8n-integration/)

---

[← Back to Tutorial Home](../README.md)
