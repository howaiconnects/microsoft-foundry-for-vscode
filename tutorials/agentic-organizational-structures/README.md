# Agentic Organizational Structures Tutorial

Welcome to the comprehensive guide on building agentic organizational structures with Microsoft AI Foundry. This tutorial is designed for users who have been using AI Foundry but haven't explored the agents and workflows features yet.

## 📚 What You'll Learn

This tutorial provides a progressive, step-by-step approach to building increasingly complex multi-agent systems, from a single agent to sophisticated team-based hierarchies with orchestration layers.

## 🎯 Prerequisites

- Basic familiarity with Microsoft AI Foundry
- VS Code with Microsoft Foundry extension installed
- Access to an Azure AI Foundry project
- Understanding of YAML configuration files

## 📖 Tutorial Structure

The tutorial is organized in progressive levels, each building on the previous:

### [Level 1: Single Agent Foundation](./level-1-single-agent/)
**Complexity:** ⭐ Basic  
**Concepts:** Core agent framework, basic configuration, model selection, instructions  
**Tools:** None (pure LLM agent)

Start here to understand the fundamental building blocks of AI Foundry agents.

### [Level 2: Enhanced Single Agent with Airtable Integration](./level-2-airtable-integration/)
**Complexity:** ⭐⭐ Intermediate  
**Concepts:** MCP server integration, external data access, API SDK usage  
**Tools:** MCP (Airtable), OpenAPI

Learn how to connect agents to external data sources using Airtable SDK and MCP servers.

### [Level 3: Two-Tier Agent Structure](./level-3-two-tier-structure/)
**Complexity:** ⭐⭐ Intermediate  
**Concepts:** Orchestrator-worker pattern, agent communication, task delegation  
**Tools:** MCP, File Search

Build your first multi-agent system with an orchestrator managing worker agents.

### [Level 4: Project Manager Orchestration Layer](./level-4-project-manager/)
**Complexity:** ⭐⭐⭐ Advanced  
**Concepts:** Complex orchestration, multi-agent coordination, workflow management  
**Tools:** MCP (Airtable + N8N), OpenAPI, File Search

Introduce a sophisticated Project Manager agent that coordinates multiple specialized agents.

### [Level 5: Team-Based Hierarchical Structure](./level-5-team-hierarchy/)
**Complexity:** ⭐⭐⭐⭐ Expert  
**Concepts:** Team leads, sub-agents, hierarchical orchestration, scalable architecture  
**Tools:** Full toolkit (MCP, OpenAPI, File Search, Code Interpreter, Bing)

Design enterprise-scale multi-team structures with team leads acting as middle-layer orchestrators.

### [Level 6: Full N8N Workflow Integration](./level-6-n8n-integration/)
**Complexity:** ⭐⭐⭐⭐⭐ Expert  
**Concepts:** Workflow automation, event-driven architecture, cross-system orchestration  
**Tools:** N8N integration, webhooks, automated workflows

Integrate AI agents with N8N for complete workflow automation and management.

## 🔧 Key Integrations Covered

### Airtable SDK & MCP
- Connecting agents to Airtable databases
- Reading and writing data through MCP servers
- Managing structured data in multi-agent systems

### N8N AI Workflow Automation
- Triggering workflows from agent actions
- Orchestrating complex multi-system processes
- Building event-driven agentic systems
- Managing workflow state and coordination

## 📊 Architecture Patterns

This tutorial covers several proven architectural patterns:

1. **Single Agent Pattern** - Basic autonomous agent
2. **Orchestrator-Worker Pattern** - One coordinator, multiple workers
3. **Hierarchical Pattern** - Multi-level management structure
4. **Team-Based Pattern** - Organized functional teams with leads
5. **Event-Driven Pattern** - Workflow-triggered agent execution

## 🚀 Getting Started

1. Navigate to [Level 1](./level-1-single-agent/) to begin
2. Follow each level sequentially to build understanding
3. Experiment with the provided sample configurations
4. Customize examples for your use cases

## 💡 Best Practices

- Start simple and add complexity gradually
- Test each configuration before moving to the next level
- Keep agent instructions clear and specific
- Use appropriate tools for each agent's role
- Monitor agent interactions and refine as needed

## 📚 Additional Resources

- [Microsoft AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [Agent Configuration Schema](../../schema/agent/1.0.0/schema.json)
- [Sample Agents](../../samples/agents/)

## 🤝 Contributing

Have suggestions or improvements? Please feel free to contribute to this tutorial.

---

**Ready to begin?** Start with [Level 1: Single Agent Foundation](./level-1-single-agent/) →
