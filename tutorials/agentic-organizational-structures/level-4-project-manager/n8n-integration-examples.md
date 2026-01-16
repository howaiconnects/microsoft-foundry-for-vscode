# N8N Workflow Integration Examples

This document provides practical examples of N8N workflows that integrate with AI Foundry agents in a Project Manager orchestration pattern.

## Overview

N8N workflows complement AI agents by handling:
- Multi-system orchestration
- Scheduled and recurring tasks
- Event-driven automation
- Complex integration logic
- Error recovery and retries

## Integration Pattern

```
AI Agent ←→ N8N Workflow ←→ External Systems
   ↓            ↓              ↓
Decision     Automation    Integration
Making      Orchestration   Execution
```

## Example 1: Customer Onboarding Workflow

### Workflow Name: `customer_onboarding_workflow`

### Trigger
**Webhook**: Triggered by Project Manager Agent with customer data

### Workflow Structure

```
1. [Webhook Trigger] Receive customer data
   ↓
2. [Airtable] Create customer record
   ↓
3. [Branch] Split by customer tier
   ├─→ Enterprise Path
   │   ├─→ [Email] Send premium welcome email
   │   ├─→ [Calendly] Schedule executive onboarding call
   │   └─→ [Slack] Notify enterprise team
   ├─→ Business Path
   │   ├─→ [Email] Send standard welcome email
   │   └─→ [Slack] Notify business team
   └─→ Starter Path
       ├─→ [Email] Send self-service welcome email
       └─→ [Documentation] Send getting started guide
   ↓
4. [Parallel Execution]
   ├─→ [PDF Generator] Create service agreement
   ├─→ [API Call] Provision user account
   ├─→ [Calendar] Schedule training sessions
   └─→ [Database] Create support tickets
   ↓
5. [Wait] Delay 2 hours (allow provisioning)
   ↓
6. [Conditional] Check if account provisioned
   ├─→ YES: Continue
   └─→ NO: Send alert, retry provisioning
   ↓
7. [Email] Send login credentials
   ↓
8. [Airtable] Update customer status to "Active"
   ↓
9. [Webhook] Callback to Project Manager Agent
   ↓
10. [Merge] Combine all branches
```

### N8N Configuration (JSON)

```json
{
  "name": "Customer Onboarding Workflow",
  "nodes": [
    {
      "parameters": {
        "httpMethod": "POST",
        "path": "customer-onboarding",
        "responseMode": "immediate"
      },
      "name": "Webhook Trigger",
      "type": "n8n-nodes-base.webhook",
      "position": [250, 300]
    },
    {
      "parameters": {
        "values": {
          "string": [
            {
              "name": "customer_id",
              "value": "={{ $json.customer_id }}"
            },
            {
              "name": "tier",
              "value": "={{ $json.tier }}"
            }
          ]
        }
      },
      "name": "Extract Data",
      "type": "n8n-nodes-base.set",
      "position": [450, 300]
    },
    {
      "parameters": {
        "conditions": {
          "string": [
            {
              "value1": "={{ $json.tier }}",
              "operation": "equals",
              "value2": "Enterprise"
            }
          ]
        }
      },
      "name": "Check Tier",
      "type": "n8n-nodes-base.if",
      "position": [650, 300]
    }
  ]
}
```

### Agent Integration Code

```yaml
# In Project Manager Agent instructions
When customer onboarding needed:

1. Validate customer data
2. Trigger N8N workflow:
   
   POST https://n8n.example.com/webhook/customer-onboarding
   {
     "customer_id": "cust_12345",
     "company_name": "Acme Corp",
     "contact_name": "John Doe",
     "contact_email": "john@acme.com",
     "tier": "Enterprise",
     "industry": "Technology",
     "region": "North America",
     "callback_url": "https://agent-api.example.com/onboarding-complete"
   }

3. Store workflow execution ID
4. Monitor for callback
5. Process completion results
```

## Example 2: Daily Data Sync Workflow

### Workflow Name: `daily_airtable_sync`

### Trigger
**Cron**: Runs daily at 2:00 AM UTC

### Purpose
Synchronize data between Airtable and external systems, triggered by time-based schedule.

### Workflow Structure

```
1. [Schedule Trigger] Daily at 2:00 AM
   ↓
2. [Airtable] Fetch all updated records (last 24h)
   ↓
3. [Function] Transform data format
   ↓
4. [Split in Batches] Process 100 records at a time
   ↓
5. [Loop through batches]
   ├─→ [HTTP Request] Send to external API
   ├─→ [Conditional] Check response
   │   ├─→ Success: Continue
   │   └─→ Error: Log and retry
   └─→ [Airtable] Update sync status
   ↓
6. [Aggregate] Collect results
   ↓
7. [Function] Generate summary report
   ↓
8. [Email] Send report to team
   ↓
9. [Webhook] Notify Project Manager Agent
```

### Configuration

```json
{
  "name": "Daily Airtable Sync",
  "nodes": [
    {
      "parameters": {
        "rule": {
          "interval": [
            {
              "field": "cronExpression",
              "expression": "0 2 * * *"
            }
          ]
        }
      },
      "name": "Daily Trigger",
      "type": "n8n-nodes-base.scheduleTrigger",
      "position": [250, 300]
    },
    {
      "parameters": {
        "application": "airtable",
        "operation": "list",
        "base": "={{ $env.AIRTABLE_BASE_ID }}",
        "table": "Customers",
        "filterByFormula": "LAST_MODIFIED_TIME() > DATEADD(NOW(), -1, 'days')"
      },
      "name": "Fetch Updated Records",
      "type": "n8n-nodes-base.airtable",
      "position": [450, 300]
    }
  ]
}
```

## Example 3: Event-Driven Report Generation

### Workflow Name: `generate_weekly_report`

### Trigger
**N8N Triggered by Agent**: When Project Manager Agent requests report

### Workflow Structure

```
1. [Webhook] Receive report request from agent
   ↓
2. [Parallel Data Collection]
   ├─→ [Airtable] Query sales data
   ├─→ [Airtable] Query customer data
   ├─→ [API Call] Get external metrics
   └─→ [Database] Query analytics data
   ↓
3. [Merge] Combine all data sources
   ↓
4. [Function] Calculate metrics
   - Total sales
   - Growth rate
   - Customer acquisition
   - Conversion rates
   ↓
5. [Code] Generate charts (Chart.js)
   ↓
6. [PDF Generator] Create formatted report
   ↓
7. [Parallel Distribution]
   ├─→ [Airtable] Save report record
   ├─→ [Email] Send to executives
   ├─→ [Slack] Post summary
   └─→ [Google Drive] Store report
   ↓
8. [Webhook] Callback to agent with report URL
```

## Example 4: Alert and Escalation Workflow

### Workflow Name: `sla_breach_alert`

### Trigger
**Airtable Automation**: When SLA breach detected

### Workflow Structure

```
1. [Airtable Trigger] New SLA breach record
   ↓
2. [Function] Assess severity
   - High: Executive alert
   - Medium: Manager alert
   - Low: Team notification
   ↓
3. [Switch] Branch by severity
   ├─→ HIGH SEVERITY
   │   ├─→ [PagerDuty] Create incident
   │   ├─→ [Slack] Alert executives
   │   ├─→ [Email] Send urgent email
   │   └─→ [SMS] Text on-call manager
   ├─→ MEDIUM SEVERITY
   │   ├─→ [Slack] Notify team lead
   │   └─→ [Email] Send alert email
   └─→ LOW SEVERITY
       └─→ [Slack] Team channel notification
   ↓
4. [Webhook] Notify Project Manager Agent
   ↓
5. [Wait] Wait for acknowledgment (30 min timeout)
   ↓
6. [Conditional] Was it acknowledged?
   ├─→ YES: Close alert
   └─→ NO: Escalate to next level
```

## Example 5: Customer Communication Sequence

### Workflow Name: `customer_engagement_campaign`

### Purpose
Multi-touch communication campaign triggered by customer lifecycle events

### Workflow Structure

```
1. [Webhook] Customer event trigger
   ↓
2. [Airtable] Enrich customer data
   ↓
3. [AI Node] Personalize message using GPT
   ↓
4. [Email] Send Day 0 message
   ↓
5. [Wait] 3 days
   ↓
6. [Airtable] Check engagement status
   ↓
7. [Conditional] Did they engage?
   ├─→ YES: Send success path emails
   └─→ NO: Send re-engagement sequence
   ↓
8. [Wait] 7 days
   ↓
9. [Email] Send Day 7 follow-up
   ↓
10. [Airtable] Update campaign status
   ↓
11. [Webhook] Report results to agent
```

## Integration Best Practices

### 1. Webhook Design

**Always include:**
```json
{
  "callback_url": "URL for completion notification",
  "execution_id": "unique tracking ID",
  "timeout_seconds": 3600,
  "retry_config": {
    "max_retries": 3,
    "backoff": "exponential"
  }
}
```

### 2. Error Handling

```
[Node]
  ↓
[Try-Catch]
  ├─→ Success: Continue workflow
  └─→ Error:
      ├─→ [Log Error]
      ├─→ [Retry Logic]
      └─→ [Notify Agent of Failure]
```

### 3. State Management

Store execution state in Airtable:
```
WORKFLOW_EXECUTIONS table:
- execution_id
- workflow_name
- status (running|completed|failed)
- triggered_by (agent_id)
- started_at
- completed_at
- result_data
- error_message
```

### 4. Callback Pattern

```javascript
// N8N final node - Webhook callback
const callbackUrl = $json.callback_url;
const results = {
  execution_id: $json.execution_id,
  status: 'completed',
  data: $json.results,
  completed_at: new Date().toISOString()
};

return {
  method: 'POST',
  url: callbackUrl,
  body: results
};
```

### 5. Monitoring and Logging

Every workflow should include:
- Start/end timestamps
- Success/failure metrics
- Error logging
- Performance metrics
- Callback confirmations

## Agent-N8N Communication Protocol

### Agent Triggers Workflow

```yaml
POST /webhook/{workflow_name}
Headers:
  Content-Type: application/json
  Authorization: Bearer {token}
Body:
  {
    "execution_id": "unique_id",
    "parameters": {...},
    "callback_url": "https://agent-api/callback",
    "priority": "high|medium|low",
    "timeout": 3600
  }
```

### N8N Calls Back to Agent

```yaml
POST {callback_url}
Headers:
  Content-Type: application/json
Body:
  {
    "execution_id": "unique_id",
    "status": "completed|failed",
    "results": {...},
    "errors": [...],
    "execution_time_ms": 5432,
    "completed_at": "2024-01-15T10:30:00Z"
  }
```

## Security Considerations

1. **Authentication**: Use API keys or OAuth tokens
2. **Encryption**: HTTPS for all communications
3. **Secrets Management**: Store credentials in N8N credentials store
4. **Rate Limiting**: Implement rate limits on webhooks
5. **Validation**: Validate all incoming webhook data
6. **Logging**: Log but don't expose sensitive data

## Performance Tips

1. **Parallel Processing**: Use parallel branches for independent operations
2. **Batch Operations**: Process records in batches (50-100 at a time)
3. **Timeouts**: Set reasonable timeouts for long-running operations
4. **Caching**: Cache frequently accessed data
5. **Async Operations**: Use async execution for non-blocking workflows

## Testing Workflows

### Manual Testing
1. Use N8N webhook test feature
2. Send sample payloads
3. Verify each node execution
4. Check callback responses

### Integration Testing
1. Deploy workflow to test environment
2. Trigger from test agent
3. Verify end-to-end flow
4. Monitor for errors

### Production Monitoring
1. Track success/failure rates
2. Monitor execution times
3. Alert on failures
4. Review error logs

## Resources

- [N8N Documentation](https://docs.n8n.io/)
- [N8N Webhook Nodes](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)
- [N8N Best Practices](https://docs.n8n.io/hosting/scaling/)
- [Airtable N8N Integration](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.airtable/)

---

[← Back to Level 4](./README.md)
