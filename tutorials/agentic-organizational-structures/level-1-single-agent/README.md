# Level 1: Single Agent Foundation

## 🎯 Learning Objectives

In this level, you'll learn:
- The fundamental structure of an AI Foundry agent
- How agents are configured using YAML
- The role of model selection and parameters
- Writing effective agent instructions
- Understanding the agent lifecycle

## 📖 Overview

A single agent is the foundation of all agentic systems. Before building complex multi-agent architectures, it's crucial to understand how individual agents work.

## 🏗️ Agent Framework Architecture

### Core Components

An AI Foundry agent consists of these essential components:

```
┌─────────────────────────────────────────┐
│           Agent Configuration            │
├─────────────────────────────────────────┤
│  1. Identity (name, description, id)    │
│  2. Model Configuration                  │
│  3. Instructions (system prompt)         │
│  4. Tools (optional)                     │
│  5. Metadata (authors, tags)             │
└─────────────────────────────────────────┘
```

### 1. Identity Layer
- **Name**: Human-readable identifier for the agent
- **Description**: Purpose and capabilities of the agent
- **ID**: Unique Azure-managed identifier (auto-generated)

### 2. Model Layer
The model defines the AI's cognitive capabilities:
- **Model ID**: Which LLM to use (e.g., gpt-4o, gpt-4.1)
- **Temperature**: Creativity vs. consistency (0-1)
  - Low (0-0.3): Focused, deterministic responses
  - Medium (0.4-0.7): Balanced creativity
  - High (0.8-1.0): Creative, varied responses
- **Top P**: Alternative sampling method (nucleus sampling)

### 3. Instructions Layer
The system prompt that defines:
- Agent's role and personality
- Task-specific guidelines
- Response format requirements
- Behavioral constraints
- Domain expertise

### 4. Tools Layer (Optional)
Extensions that give agents capabilities beyond language:
- File search for retrieval
- Code interpreter for computation
- Bing grounding for web search
- OpenAPI for external APIs
- MCP for custom integrations

### 5. Metadata Layer
Organizational information:
- Authors and ownership
- Tags for categorization
- Version tracking

## 📝 Configuration Walkthrough

Let's examine the basic agent configuration:

```yaml
# yaml-language-server: $schema=https://aka.ms/ai-foundry-vsc/agent/1.0.0
version: 1.0.0
name: Research Assistant
description: A specialized agent for conducting research and summarizing information
id: 
metadata:
  authors:
    - Your Name
  tags:
    - research
    - analysis
model:
  id: gpt-4o
  options:
    temperature: 0.3
    top_p: 0.9
instructions: |
  You are a professional research assistant specializing in information gathering and analysis.
  
  Your responsibilities:
  - Conduct thorough research on assigned topics
  - Provide accurate, well-sourced information
  - Summarize complex information clearly
  - Maintain objectivity and cite sources when possible
  
  Communication style:
  - Clear and concise
  - Professional but approachable
  - Well-structured responses with headings and bullet points
  
  Always prioritize accuracy over speed.
```

### Breaking Down Each Section

#### Version
```yaml
version: 1.0.0
```
The schema version ensures compatibility with the AI Foundry platform.

#### Identity
```yaml
name: Research Assistant
description: A specialized agent for conducting research and summarizing information
id: 
```
- Name is descriptive and indicates purpose
- Description expands on capabilities
- ID is left empty for Azure to auto-generate on deployment

#### Metadata
```yaml
metadata:
  authors:
    - Your Name
  tags:
    - research
    - analysis
```
Helps organize and track agents in large deployments.

#### Model Configuration
```yaml
model:
  id: gpt-4o
  options:
    temperature: 0.3
    top_p: 0.9
```
- **gpt-4o**: Latest optimized GPT-4 model
- **temperature: 0.3**: Low value for consistent, factual research
- **top_p: 0.9**: Slightly reduced to maintain focus

#### Instructions
```yaml
instructions: |
  You are a professional research assistant...
```
This is the agent's "constitution" - the system prompt that defines behavior, capabilities, and constraints.

## 🎨 Design Principles

### 1. Single Responsibility Principle
Each agent should have one clear purpose. Don't try to make a single agent do everything.

**Good Example:**
- Research Assistant (focused on research)
- Data Analyst (focused on analysis)
- Content Writer (focused on writing)

**Poor Example:**
- General Purpose Agent (tries to do research, analysis, writing, coding, etc.)

### 2. Clear Instructions
Write instructions that are:
- **Specific**: Define exact behaviors
- **Structured**: Use sections and bullet points
- **Comprehensive**: Cover edge cases
- **Contextual**: Include relevant domain knowledge

### 3. Appropriate Model Selection
Choose models based on task requirements:
- **gpt-4o**: Best for complex reasoning, latest features
- **gpt-4.1**: Solid general-purpose performance
- **gpt-3.5-turbo**: Cost-effective for simpler tasks

### 4. Temperature Tuning
Adjust temperature based on task type:
- **Factual/Analytical**: 0.0-0.3
- **Balanced**: 0.4-0.7
- **Creative**: 0.8-1.0

## 🚀 Deployment Process

### Step 1: Create Configuration File
Save your agent configuration as a `.agent.yaml` file.

### Step 2: Deploy via VS Code
1. Open the file in VS Code
2. Use Command Palette (Ctrl+Shift+P)
3. Run "Microsoft Foundry: Deploy Agent"
4. Select your target project

### Step 3: Verify Deployment
Check the Azure Resources panel to confirm the agent appears in your project.

### Step 4: Test the Agent
Interact with your agent through:
- The VS Code interface
- Azure AI Foundry portal
- API calls from your application

## 💡 Use Cases for Single Agents

### 1. Customer Support Agent
```yaml
name: Customer Support Agent
model:
  id: gpt-4o
  options:
    temperature: 0.5
instructions: |
  You are a helpful customer support agent for [Company Name].
  - Always maintain a friendly, professional tone
  - Prioritize customer satisfaction
  - Escalate complex issues appropriately
  - Provide step-by-step guidance
```

### 2. Code Review Agent
```yaml
name: Code Review Assistant
model:
  id: gpt-4o
  options:
    temperature: 0.2
instructions: |
  You are an expert code reviewer specializing in [Language].
  - Focus on code quality, security, and best practices
  - Provide constructive feedback with examples
  - Suggest improvements with rationale
  - Be thorough but concise
```

### 3. Content Moderator Agent
```yaml
name: Content Moderator
model:
  id: gpt-4o
  options:
    temperature: 0.1
instructions: |
  You are a content moderation agent responsible for reviewing user-generated content.
  - Flag content that violates community guidelines
  - Classify violations by severity
  - Provide clear reasoning for decisions
  - Maintain consistency in moderation standards
```

## 📊 Performance Considerations

### Response Quality
- Clear instructions = Better responses
- Appropriate temperature = More predictable behavior
- Specific examples = Improved understanding

### Cost Optimization
- Use appropriate model tiers for tasks
- Set reasonable token limits
- Consider caching for repeated queries

### Scalability
Single agents scale well for:
- Independent tasks
- Parallel processing of similar requests
- Stateless operations

## 🔍 Testing Your Agent

### Test Scenarios
Create test cases covering:
1. **Typical requests**: Common use cases
2. **Edge cases**: Unusual or challenging inputs
3. **Error conditions**: Invalid or malformed requests
4. **Performance**: Response time and quality

### Evaluation Criteria
- **Accuracy**: Does it provide correct information?
- **Consistency**: Similar inputs → similar outputs?
- **Completeness**: Are responses thorough?
- **Tone**: Does it match the intended personality?

## 🎓 Key Takeaways

1. **Foundation First**: Understanding single agents is crucial before building multi-agent systems
2. **Clear Purpose**: Each agent should have a well-defined role
3. **Effective Instructions**: Detailed system prompts drive agent behavior
4. **Model Selection**: Choose the right model for the task
5. **Iterative Refinement**: Test and improve based on results

## 📁 Sample Configuration

See [`basic-agent.agent.yaml`](./basic-agent.agent.yaml) for a complete working example.

## ➡️ Next Steps

Ready for more complexity? Proceed to [Level 2: Enhanced Single Agent with Airtable Integration](../level-2-airtable-integration/)

In the next level, you'll learn how to:
- Add external tools to agents
- Integrate with Airtable using MCP servers
- Access and manipulate external data
- Use OpenAPI specifications for custom integrations

---

[← Back to Tutorial Home](../README.md) | [Next Level →](../level-2-airtable-integration/)
