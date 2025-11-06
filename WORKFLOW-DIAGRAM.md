```mermaid
%%{init: {'theme':'base', 'themeVariables': { 'primaryColor':'#4DD0E1','primaryTextColor':'#000','primaryBorderColor':'#0097A7','lineColor':'#757575','secondaryColor':'#ff8fab','tertiaryColor':'#7EE787'}}}%%
graph TD
    startAgentflow_0([Start])
    style startAgentflow_0 fill:#7EE787,stroke:#333,stroke-width:2px
    stickyNote_purpose[Sticky Note]
    style stickyNote_purpose fill:#fee440,stroke:#333,stroke-width:2px
    agent_planner[Agent.Planner]
    style agent_planner fill:#4DD0E1,stroke:#333,stroke-width:2px
    stickyNote_iteration[Sticky Note]
    style stickyNote_iteration fill:#9C89B8,stroke:#333,stroke-width:2px
    iterationAgentflow_0[For Each Loop]
    style iterationAgentflow_0 fill:#9C89B8,stroke:#333,stroke-width:2px
    agent_processor[Agent.Processor]
    style agent_processor fill:#4DD0E1,stroke:#333,stroke-width:2px
    stickyNote_aggregation[Sticky Note]
    style stickyNote_aggregation fill:#4DD0E1,stroke:#333,stroke-width:2px
    agent_aggregator[Agent.Aggregator]
    style agent_aggregator fill:#4DD0E1,stroke:#333,stroke-width:2px
    directReplyAgentflow_0[Direct Reply]
    style directReplyAgentflow_0 fill:#FFA726,stroke:#333,stroke-width:2px

    startAgentflow_0 --> agent_planner
    agent_planner --> iterationAgentflow_0
    iterationAgentflow_0 --> agent_processor
    agent_processor --> agent_aggregator
    agent_aggregator --> directReplyAgentflow_0
```

![Nodes](https://img.shields.io/badge/Nodes-9-blue) ![Agents](https://img.shields.io/badge/Agents-3-green) ![Complexity](https://img.shields.io/badge/Complexity-Complex-orange) ![Memory](https://img.shields.io/badge/Memory-Enabled-purple) ![Tools](https://img.shields.io/badge/Tools-0-red)

---

**Total Nodes**: 9 | 
**Agents**: 3 | 
**Complexity**: Complex

<details>
<summary><b>🔍 View Agent Details (Click to Expand)</b></summary>

| Agent | Type | Description |
|-------|------|-------------|
| 🚀 Start | Start | Starting point of the agentflow |
| 📝 Sticky Note | StickyNote | Add notes to the agent flow |
| 🤖 Agent.Planner | Agent | Dynamically choose and utilize tools during run... |
| 📝 Sticky Note | StickyNote | Add notes to the agent flow |
| 🔁 For Each Loop | Iteration | Execute the nodes within the iteration block fo... |
| 🤖 Agent.Processor | Agent | Dynamically choose and utilize tools during run... |
| 📝 Sticky Note | StickyNote | Add notes to the agent flow |
| 🤖 Agent.Aggregator | Agent | Dynamically choose and utilize tools during run... |
| 💭 Direct Reply | DirectReply | Reply directly without agent processing |

</details>


### 🎨 Node Type Legend

| Icon | Type | Description |
|------|------|-------------|
| 🚀 | Start | Entry point of the workflow |
| 🤖 | Agent | AI agent with reasoning capabilities |
| 💬 | LLM | Large Language Model node |
| 🔀 | Condition | Branching logic |
| 🎯 | ConditionAgent | AI-powered conditional routing |
| 🔧 | Tool | External tool integration |
| ▶️ | ExecuteFlow | Execute another workflow |
| ⚙️ | CustomFunction | Custom JavaScript function |
| 🌐 | HTTP | HTTP request node |
| 👤 | HumanInput | Request human input/approval |
| 💭 | DirectReply | Direct message response |
| 🔄 | Loop | Loop back to previous node |
| 🔁 | Iteration | Iterate over array |
| 📝 | StickyNote | Documentation note |
