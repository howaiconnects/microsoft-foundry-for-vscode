# Quick Start Guide

Get started with agentic organizational structures in minutes!

## 🚀 5-Minute Quick Start

### Step 1: Choose Your Starting Level

**New to agents?** → Start with [Level 1: Single Agent](./level-1-single-agent/)

**Have basic agent experience?** → Jump to [Level 2: Airtable Integration](./level-2-airtable-integration/)

**Ready for multi-agent?** → Go to [Level 3: Two-Tier Structure](./level-3-two-tier-structure/)

### Step 2: Copy a Sample Configuration

Choose a sample that matches your use case:

**For research/analysis:**
```bash
cp level-1-single-agent/basic-agent.agent.yaml my-agent.agent.yaml
```

**For data management:**
```bash
cp level-2-airtable-integration/airtable-mcp-agent.agent.yaml my-agent.agent.yaml
```

**For project coordination:**
```bash
cp level-4-project-manager/project-manager-agent.agent.yaml my-agent.agent.yaml
```

### Step 3: Customize the Configuration

Edit `my-agent.agent.yaml`:

1. **Update the name**:
   ```yaml
   name: My Custom Agent
   ```

2. **Modify the description**:
   ```yaml
   description: What your agent does
   ```

3. **Adjust instructions** for your specific use case

4. **Configure tools** if needed (MCP servers, APIs, etc.)

### Step 4: Deploy

In VS Code:
1. Open `my-agent.agent.yaml`
2. Press `Ctrl+Shift+P` (Windows/Linux) or `Cmd+Shift+P` (Mac)
3. Type "Microsoft Foundry: Deploy Agent"
4. Select your target project
5. Wait for deployment confirmation

### Step 5: Test

Test your agent:
1. Find your agent in Azure Resources panel
2. Click to open chat interface
3. Send a test message
4. Verify the response

🎉 **Congratulations!** You've deployed your first agent!

---

## 📋 Common Use Cases

### Use Case 1: Customer Support Bot

**Level**: 1 (Single Agent)  
**Time**: 10 minutes  
**Components**: 1 agent, optional Bing search

**Quick Setup**:
```yaml
name: Customer Support Agent
model:
  id: gpt-4o
  options:
    temperature: 0.5
instructions: |
  You are a helpful customer support agent for [Company].
  - Answer questions about products and services
  - Provide troubleshooting help
  - Escalate complex issues
  - Maintain a friendly, professional tone
tools:
  - type: bing_grounding
    options:
      tool_connections:
        - ${BING_CONNECTION_ID}
```

---

### Use Case 2: Data Analysis Pipeline

**Level**: 3 (Two-Tier)  
**Time**: 30 minutes  
**Components**: 1 orchestrator + 3 workers

**Agent Stack**:
1. Orchestrator (coordinates workflow)
2. Data Retrieval Worker (fetches from Airtable)
3. Analysis Worker (performs calculations)
4. Report Worker (generates output)

**Start**: Use configurations from [level-3-two-tier-structure](./level-3-two-tier-structure/)

---

### Use Case 3: Enterprise CRM

**Level**: 5 (Hierarchical)  
**Time**: 2-4 hours  
**Components**: 1 executive + 4 team leads + 20 workers

**Team Structure**:
- Executive Orchestrator
- Data Team (5 agents)
- Analytics Team (5 agents)
- Operations Team (5 agents)
- Customer Success Team (5 agents)

**Start**: Use configurations from [level-5-team-hierarchy](./level-5-team-hierarchy/)

---

## 🔧 Setup Checklist

### Prerequisites
- [ ] Microsoft AI Foundry project created
- [ ] VS Code with Foundry extension installed
- [ ] Azure subscription with access
- [ ] Default project selected in VS Code

### For Airtable Integration (Levels 2+)
- [ ] Airtable account created
- [ ] Base(s) created with appropriate tables
- [ ] API key generated
- [ ] MCP server configured (if using)

### For N8N Integration (Levels 4+)
- [ ] N8N instance deployed
- [ ] Webhook URLs configured
- [ ] Workflows created
- [ ] Test webhooks working

---

## 🐛 Troubleshooting

### Agent Won't Deploy

**Error**: "Model not found"
- **Solution**: Verify model ID is correct for your project
- Check available models in AI Foundry portal

**Error**: "Invalid tool configuration"
- **Solution**: Check MCP server URL is accessible
- Verify tool IDs are correct
- Ensure authentication is configured

### Agent Not Responding as Expected

**Issue**: Responses are inconsistent
- **Solution**: Lower temperature (0.1-0.3) for more consistent behavior
- Make instructions more specific
- Add examples in instructions

**Issue**: Agent doesn't use tools
- **Solution**: Explicitly mention tools in instructions
- Provide examples of when to use each tool
- Ensure tools are properly configured

### Performance Issues

**Issue**: Slow responses
- **Solution**: Check MCP server performance
- Simplify instructions
- Reduce tool complexity
- Consider caching frequently accessed data

**Issue**: High costs
- **Solution**: Use appropriate model tier (not always gpt-4o)
- Optimize instructions to be more concise
- Implement caching strategies
- Monitor token usage

---

## 📖 Learning Path

### Week 1: Foundations
- [ ] Complete Level 1 tutorial
- [ ] Deploy and test a single agent
- [ ] Experiment with different instructions
- [ ] Try different temperature settings

### Week 2: Integration
- [ ] Complete Level 2 tutorial
- [ ] Set up Airtable integration
- [ ] Create agents that access external data
- [ ] Test data validation and CRUD operations

### Week 3: Orchestration
- [ ] Complete Level 3 tutorial
- [ ] Build your first multi-agent system
- [ ] Practice task delegation
- [ ] Implement error handling

### Week 4: Advanced Patterns
- [ ] Study Level 4 (Project Manager)
- [ ] Explore N8N integration
- [ ] Review Level 5 (Hierarchical)
- [ ] Plan your production system

---

## 💡 Pro Tips

### Tip 1: Start Small
Don't jump to Level 5 immediately. Each level builds on previous knowledge.

### Tip 2: Test Incrementally
Deploy and test after each change. Don't make multiple changes at once.

### Tip 3: Use Version Control
Store your agent configurations in Git. Track changes over time.

### Tip 4: Monitor Performance
Set up basic logging from day one. You'll need it for debugging.

### Tip 5: Document Your System
Write down your agent architecture and workflows. Future you will thank you.

### Tip 6: Learn from Examples
Study the sample configurations. They contain best practices.

### Tip 7: Join the Community
Engage with other AI Foundry users. Share learnings and get help.

---

## 🔗 Quick Links

**Documentation**:
- [Tutorial Home](./README.md)
- [Design Patterns](./DESIGN_PATTERNS.md)
- [Agent Schema](https://aka.ms/ai-foundry-vsc/agent/1.0.0)

**Levels**:
- [Level 1: Single Agent](./level-1-single-agent/)
- [Level 2: Airtable Integration](./level-2-airtable-integration/)
- [Level 3: Two-Tier Structure](./level-3-two-tier-structure/)
- [Level 4: Project Manager](./level-4-project-manager/)
- [Level 5: Team Hierarchy](./level-5-team-hierarchy/)
- [Level 6: N8N Integration](./level-6-n8n-integration/)

**External Resources**:
- [AI Foundry Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [N8N Documentation](https://docs.n8n.io/)
- [Airtable API](https://airtable.com/developers/web/api/introduction)

---

## 🆘 Getting Help

**Issues with Tutorial**:
- Check the specific level's README for details
- Review [Design Patterns](./DESIGN_PATTERNS.md)
- Search existing GitHub issues

**Issues with AI Foundry**:
- [Official Documentation](https://learn.microsoft.com/azure/ai-studio/)
- [GitHub Issues](https://github.com/microsoft/microsoft-foundry-for-vscode/issues)
- Email: vscai-support@microsoft.com

**Community Support**:
- Microsoft AI Community forums
- Stack Overflow (tag: azure-ai-foundry)

---

**Ready to build something amazing?** → [Start with Level 1](./level-1-single-agent/)
