# Level 3: Two-Tier Agent Structure

## 🎯 Learning Objectives

In this level, you'll learn:
- Orchestrator-worker architecture pattern
- How multiple agents coordinate and communicate
- Task delegation and specialization
- Managing agent workflows
- Inter-agent dependencies and handoffs

## 📖 Overview

Single agents have limits. Complex workflows often require different capabilities working in concert. The **two-tier structure** introduces specialization: an orchestrator agent that coordinates multiple worker agents, each specialized for specific tasks.

## 🏗️ Two-Tier Architecture

```
                  ┌─────────────────────────┐
                  │   Orchestrator Agent    │
                  │   (Coordinator/Router)   │
                  └───────────┬─────────────┘
                              │
                 ┌────────────┼────────────┐
                 │            │            │
                 ▼            ▼            ▼
         ┌──────────┐  ┌──────────┐  ┌──────────┐
         │ Worker   │  │ Worker   │  │ Worker   │
         │ Agent 1  │  │ Agent 2  │  │ Agent 3  │
         │          │  │          │  │          │
         │ (Data)   │  │(Analysis)│  │ (Report) │
         └──────────┘  └──────────┘  └──────────┘
              │              │              │
              ▼              ▼              ▼
          Airtable      File Search   Code Interpreter
```

## 🎭 Agent Roles

### Orchestrator Agent

**Purpose**: Task routing, coordination, and workflow management

**Responsibilities**:
- Receive user requests
- Analyze requirements
- Determine which worker agents to engage
- Route tasks to appropriate workers
- Aggregate results from multiple workers
- Provide unified response to user

**Key Characteristics**:
- Broad understanding of available agents
- Decision-making capabilities
- Workflow coordination logic
- Result synthesis abilities

### Worker Agents

**Purpose**: Specialized task execution

**Responsibilities**:
- Execute specific, well-defined tasks
- Provide results back to orchestrator
- Maintain focus on their specialty
- Handle errors in their domain

**Key Characteristics**:
- Deep expertise in specific domain
- Optimized for their task type
- Independent operation
- Clear input/output contracts

## 📝 Configuration Patterns

### Pattern 1: Orchestrator Configuration

```yaml
# yaml-language-server: $schema=https://aka.ms/ai-foundry-vsc/agent/1.0.0
version: 1.0.0
name: Workflow Orchestrator
description: Coordinates multiple specialized worker agents to handle complex workflows
id: 
metadata:
  authors:
    - AI Foundry Tutorial
  tags:
    - orchestrator
    - coordinator
    - multi-agent
model:
  id: gpt-4o
  options:
    temperature: 0.4
    top_p: 0.95
instructions: |
  You are a workflow orchestrator managing a team of specialized agents.
  
  Available worker agents:
  
  1. DATA_RETRIEVAL_AGENT
     - Capability: Query and fetch data from Airtable
     - Use when: User needs to retrieve or search for data
     - Input: Query parameters, filters, table names
     - Output: Retrieved records, data summaries
  
  2. DATA_ANALYSIS_AGENT
     - Capability: Analyze data, generate insights, statistics
     - Use when: User needs analysis, trends, or insights
     - Input: Dataset or data reference
     - Output: Analysis report, charts, insights
  
  3. REPORT_GENERATION_AGENT
     - Capability: Create formatted reports and documents
     - Use when: User needs formal documentation
     - Input: Data and formatting requirements
     - Output: Formatted report
  
  Your workflow:
  
  1. UNDERSTAND REQUEST
     - Parse user intent
     - Identify required agents
     - Determine task sequence
  
  2. PLAN EXECUTION
     - Create step-by-step plan
     - Identify dependencies between tasks
     - Allocate tasks to appropriate agents
  
  3. COORDINATE EXECUTION
     - Route tasks to worker agents
     - Track completion status
     - Handle errors or failures
  
  4. SYNTHESIZE RESULTS
     - Collect outputs from all workers
     - Combine and format results
     - Present unified response
  
  Communication protocol:
  - Format delegated tasks clearly with all required parameters
  - Track which agents have been engaged
  - Handle partial failures gracefully
  - Provide status updates for long-running workflows
  
  When delegating:
  - Be explicit about what each agent should do
  - Include all necessary context and data
  - Specify expected output format
  - Set priority levels if needed
tools:
  - type: file_search
    options:
      vector_store_ids:
        - ${AGENT_REGISTRY_VECTOR_STORE}
```

### Pattern 2: Worker Agent Configuration

```yaml
# yaml-language-server: $schema=https://aka.ms/ai-foundry-vsc/agent/1.0.0
version: 1.0.0
name: Data Retrieval Agent
description: Specialized worker agent for querying and retrieving data from Airtable
id: 
metadata:
  authors:
    - AI Foundry Tutorial
  tags:
    - worker
    - data-retrieval
    - airtable
model:
  id: gpt-4o
  options:
    temperature: 0.1
    top_p: 0.9
instructions: |
  You are a data retrieval specialist. Your sole focus is fetching data from Airtable.
  
  Your tasks:
  - Execute data queries efficiently
  - Apply filters and sorting
  - Handle pagination for large datasets
  - Validate query parameters
  - Return well-structured results
  
  Input format you expect:
  {
    "table": "table_name",
    "filters": [...],
    "sort": {...},
    "fields": [...]
  }
  
  Output format:
  {
    "status": "success|error",
    "records_count": number,
    "data": [...],
    "error": "error message if any"
  }
  
  Rules:
  - Focus only on data retrieval
  - Don't analyze or interpret data
  - Don't create or modify records
  - Report errors clearly
  - Optimize for query performance
tools:
  - type: mcp
    id: airtable_server
    options:
      server_url: https://your-mcp-server.example.com/airtable
      allowed_tools:
        - query_records
        - get_table
        - get_table_schema
```

## 🔄 Workflow Patterns

### Pattern A: Sequential Workflow

Tasks executed one after another, where each depends on the previous:

```
User Request
     ↓
Orchestrator
     ↓
[1] Data Retrieval → [2] Data Analysis → [3] Report Generation
     ↓                    ↓                    ↓
  Results          ←  Results          ←   Final Report
     ↓
Orchestrator
     ↓
User Response
```

**Example Use Case**: "Generate a sales report for Q4"
1. Retrieve Q4 sales data from Airtable
2. Analyze trends and calculate metrics
3. Generate formatted report

### Pattern B: Parallel Workflow

Multiple workers execute simultaneously for independent tasks:

```
User Request
     ↓
Orchestrator
     ↓
     ├──→ [1] Data Retrieval (Customer data)
     ├──→ [2] Data Retrieval (Sales data)
     └──→ [3] Data Retrieval (Product data)
         ↓         ↓         ↓
         └─────────┼─────────┘
                   ↓
              Orchestrator (Combine)
                   ↓
              User Response
```

**Example Use Case**: "Show me a dashboard of all key metrics"
1. Fetch customer statistics (parallel)
2. Fetch sales statistics (parallel)
3. Fetch product performance (parallel)
4. Combine into unified dashboard

### Pattern C: Conditional Workflow

Decision points determine which agents to engage:

```
User Request
     ↓
Orchestrator (Analyze Request)
     ↓
Is Complex Analysis Needed?
     ├─ YES → Engage Analysis Agent
     └─ NO → Proceed with simple retrieval
          ↓
     Orchestrator
          ↓
     User Response
```

## 💡 Real-World Use Case: Sales Pipeline Manager

Let's build a complete two-tier system for managing a sales pipeline.

### Orchestrator: Sales Pipeline Manager

```yaml
name: Sales Pipeline Orchestrator
description: Manages sales pipeline operations by coordinating specialized agents
instructions: |
  You coordinate a sales pipeline management system with these worker agents:
  
  LEAD_RETRIEVAL_AGENT:
  - Fetches lead data from Airtable CRM
  - Filters and searches leads
  
  LEAD_SCORING_AGENT:
  - Analyzes lead quality
  - Calculates lead scores
  - Identifies hot prospects
  
  COMMUNICATION_AGENT:
  - Generates outreach messages
  - Personalizes communication
  - Tracks engagement
  
  REPORTING_AGENT:
  - Creates pipeline reports
  - Tracks conversion metrics
  - Generates forecasts
  
  Common workflows:
  
  1. "Show me high-priority leads"
     → LEAD_RETRIEVAL_AGENT (filter: priority=high)
     → Present results
  
  2. "Score all new leads from this week"
     → LEAD_RETRIEVAL_AGENT (filter: created_this_week)
     → LEAD_SCORING_AGENT (input: retrieved leads)
     → Update scores in Airtable
  
  3. "Generate weekly pipeline report"
     → LEAD_RETRIEVAL_AGENT (all active leads)
     → REPORTING_AGENT (input: lead data)
     → Format and present report
  
  Coordinate efficiently and provide clear status updates.
```

### Worker 1: Lead Retrieval Agent

```yaml
name: Lead Retrieval Agent
description: Fetches and filters lead data from Airtable CRM
instructions: |
  You retrieve lead information from the Airtable CRM.
  
  Tables you access:
  - Leads: id, name, email, company, status, priority, score, source, created_at
  - Activities: id, lead_id, type, date, notes
  - Communications: id, lead_id, message, sent_at, response
  
  Support these operations:
  - Get all leads with filters
  - Get single lead by ID
  - Search leads by criteria
  - Get lead activity history
  
  Always return complete lead records with all relevant fields.
tools:
  - type: mcp
    id: airtable_crm
    options:
      server_url: https://crm-mcp-server.example.com
      allowed_tools:
        - query_records
        - get_record
```

### Worker 2: Lead Scoring Agent

```yaml
name: Lead Scoring Agent
description: Analyzes and scores leads based on multiple factors
instructions: |
  You analyze leads and calculate quality scores (0-100).
  
  Scoring factors:
  - Company size (+20 if enterprise, +10 if mid-market)
  - Engagement level (+15 for email opens, +25 for meetings)
  - Budget indicated (+20 if qualified, +10 if potential)
  - Timeline (+15 if urgent, +5 if 3-6 months)
  - Authority (+20 if decision maker, +10 if influencer)
  
  Output format:
  {
    "lead_id": "...",
    "score": 85,
    "category": "Hot|Warm|Cold",
    "factors": ["High engagement", "Decision maker"],
    "recommendation": "Priority follow-up recommended"
  }
  
  You use code interpreter to calculate scores efficiently.
tools:
  - type: code_interpreter
    options:
      file_ids: []
```

### Worker 3: Reporting Agent

```yaml
name: Sales Reporting Agent
description: Generates formatted sales pipeline reports
instructions: |
  You create professional sales pipeline reports.
  
  Report types:
  1. Pipeline Overview: Total leads by stage, conversion rates
  2. Performance Report: Lead sources, close rates, time-to-close
  3. Forecast Report: Projected revenue, probability-weighted pipeline
  
  Reports include:
  - Executive summary
  - Key metrics and KPIs
  - Visualizations (described textually)
  - Trends and insights
  - Actionable recommendations
  
  Format reports professionally with clear sections and tables.
tools:
  - type: code_interpreter
    options:
      file_ids: []
```

## 🔧 Implementation Strategy

### Step 1: Design Agent Hierarchy

1. **Identify the main workflow** you need to support
2. **Break down into logical tasks** that can be specialized
3. **Design orchestrator responsibilities** (routing, coordination)
4. **Define worker agent specializations** (clear, narrow focus)

### Step 2: Define Communication Protocol

```yaml
# Standard task delegation format
task_delegation:
  agent: "WORKER_AGENT_NAME"
  task: "Specific task description"
  input:
    # Structured input data
  parameters:
    # Task parameters
  priority: "high|medium|low"
  
# Standard response format
task_response:
  status: "success|error|partial"
  agent: "WORKER_AGENT_NAME"
  output:
    # Result data
  errors: []
  metadata:
    execution_time: "..."
```

### Step 3: Deploy Agents

1. Deploy all worker agents first
2. Test each worker independently
3. Deploy orchestrator agent
4. Test end-to-end workflows

### Step 4: Monitor and Refine

- Track success rates
- Identify bottlenecks
- Refine agent instructions
- Optimize task routing

## 🧪 Testing Two-Tier Systems

### Test Scenarios

1. **Single Worker Engagement**
   - Orchestrator routes to one worker
   - Verify correct agent selection
   - Validate result handling

2. **Sequential Workflow**
   - Multi-step process
   - Verify data passing between workers
   - Test error propagation

3. **Parallel Execution**
   - Multiple workers simultaneously
   - Verify result aggregation
   - Test partial failures

4. **Error Handling**
   - Worker failure scenarios
   - Invalid inputs
   - Timeout handling

### Monitoring Checklist

- [ ] Orchestrator correctly identifies required workers
- [ ] Tasks delegated with complete information
- [ ] Workers execute tasks independently
- [ ] Results properly aggregated
- [ ] Errors handled gracefully
- [ ] User receives coherent responses

## 📊 Performance Considerations

### Latency
- Orchestration adds overhead
- Parallel execution reduces total time
- Cache frequent queries

### Cost
- Multiple agent invocations increase cost
- Optimize by minimizing unnecessary calls
- Use efficient worker agents (lower-tier models when possible)

### Scalability
- Workers can scale independently
- Orchestrator becomes potential bottleneck
- Consider load balancing for high volume

## 🎓 Key Takeaways

1. **Separation of Concerns**: Orchestrator coordinates, workers execute
2. **Specialization**: Workers focused on specific tasks perform better
3. **Clear Contracts**: Well-defined inputs/outputs between agents
4. **Error Handling**: Graceful degradation when workers fail
5. **Flexibility**: Easy to add new workers without changing others

## 📁 Sample Configurations

- [`orchestrator-agent.agent.yaml`](./orchestrator-agent.agent.yaml) - Workflow coordinator
- [`data-retrieval-worker.agent.yaml`](./data-retrieval-worker.agent.yaml) - Data fetching specialist
- [`analysis-worker.agent.yaml`](./analysis-worker.agent.yaml) - Data analysis specialist  
- [`reporting-worker.agent.yaml`](./reporting-worker.agent.yaml) - Report generation specialist

## ➡️ Next Steps

Ready for more sophisticated orchestration? Proceed to [Level 4: Project Manager Orchestration Layer](../level-4-project-manager/)

In the next level, you'll learn:
- Advanced orchestration with resource management
- Project Manager agent pattern
- Complex workflow coordination
- Integration with N8N for workflow automation
- State management across workflows

---

[← Previous Level](../level-2-airtable-integration/) | [Back to Tutorial Home](../README.md) | [Next Level →](../level-4-project-manager/)
