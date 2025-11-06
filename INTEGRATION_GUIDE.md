# Integration Guide: AFv2 Pattern #7 - Batch Processing

**How to import and use this Flowise workflow**

---

## Prerequisites

### 1. Flowise Installation
- Flowise v1.8.0 or higher (required for Iteration Node)
- Running Flowise instance (local or cloud)

### 2. API Keys
- Anthropic API key (for Claude Sonnet 4.5)

### 3. Custom Tools Setup

Before importing the workflow, create these 2 custom tools in Flowise:

#### Tool 1: currentDateTime

1. Go to **Tools** → **Custom Tools** → **Add Custom Tool**
2. Configure:
   - **Name**: `currentDateTime`
   - **Description**: "Returns current date and time"
   - **Type**: JavaScript function
   - **Code**:
     ```javascript
     return {
       currentDateTime: new Date().toISOString(),
       timestamp: Date.now(),
       date: new Date().toDateString(),
       time: new Date().toTimeString()
     };
     ```
3. Save tool

#### Tool 2: searxng-search

1. Go to **Tools** → **Custom Tools** → **Add Custom Tool**
2. Configure:
   - **Name**: `searxng-search`
   - **Description**: "Federated web/meta search. Use when you need fresh facts or sources."
   - **Type**: HTTP API
   - **Base URL**: `https://s.llam.ai`
   - **Method**: GET
   - **Endpoint**: `/search`
   - **Parameters**:
     - `q` (string, required): Search query
     - `format` (string, default: "json"): Response format
3. Save tool

---

## Import Workflow

### Step 1: Download JSON

Download `07-batch-processing.json` from this repository.

### Step 2: Import to Flowise

1. Open Flowise UI in browser
2. Click **"Import Workflow"** button (top right)
3. Select `07-batch-processing.json` from your downloads
4. Wait for import to complete (should see 9 nodes on canvas)

### Step 3: Configure Credentials

1. Click on any agent node (Planner, Processor, or Aggregator)
2. Find **"Model Configuration"** section
3. Under **"Credential"**, select **"Anthropic API Key"**
4. If credential doesn't exist:
   - Click **"+ Add New Credential"**
   - Enter your Anthropic API key
   - Name it "Anthropic API Key"
   - Save
5. Repeat for all 3 agent nodes

### Step 4: Verify Connections

Check that all nodes are connected (no "not connected" errors):

✅ Start → Planner Agent
✅ Planner Agent → Iteration Node
✅ Iteration Node → Processor Agent
✅ Processor Agent → Aggregator Agent
✅ Aggregator Agent → Direct Reply

### Step 5: Save Workflow

1. Click **"Save"** button
2. Name the workflow: "AFv2 Pattern 7 - Batch Processing"
3. Confirm save

---

## Testing the Workflow

### Test 1: Basic Sentiment Analysis

1. Start a **new chat session** in Flowise
2. Send message:
   ```
   Analyze sentiment for these 3 reviews:
   - "Great product!"
   - "Not bad"
   - "Terrible quality"
   ```
3. Expected response:
   ```
   Batch processing complete. Analyzed 3 items with 100% success rate.
   Sentiment breakdown: 1 positive, 1 neutral, 1 negative.
   ```

### Test 2: Empty Array Handling

1. Send message:
   ```
   Analyze sentiment for these reviews: []
   ```
2. Expected response:
   ```
   Error: No items to process. Please provide at least one item.
   ```

### Test 3: Single Item

1. Send message:
   ```
   Analyze: ["Excellent!"]
   ```
2. Expected response:
   ```
   Batch processing complete. Analyzed 1 item with 100% success rate.
   Sentiment: positive (confidence: 0.95)
   ```

---

## Configuration Options

### Customize Batch Operation Type

To change from sentiment analysis to another operation (validation, transformation, etc.):

1. Click **Processor Agent** node
2. Edit **System Prompt** section
3. Replace sentiment analysis instructions with your operation:
   ```
   For Data Validation:
   - Check if item meets criteria (format, type, constraints)
   - Return success: true/false
   - Include validation errors if any

   Output Format:
   {
     "item": "original",
     "valid": true/false,
     "errors": ["error1", "error2"],
     "success": true
   }
   ```
4. Save changes

### Adjust Model Settings

For each agent, you can modify:
- **Temperature**: 0.0-1.0 (default: 0.7)
- **Max Tokens**: Output limit (default: 4096)
- **Max Iterations**: Tool usage limit (default: 5)

---

## Flow State Variables

The workflow uses these Flow State variables:

| Variable | Type | Description |
|----------|------|-------------|
| `$flow.state.items` | array | Items to process (set by Planner) |
| `$flow.state.processed_count` | number | Total items processed |
| `$flow.state.success_count` | number | Successful operations |
| `$flow.state.failure_count` | number | Failed operations |
| `$flow.state.iteration_results` | array | Full results from all iterations |
| `$flow.state.batch_summary` | object | Aggregated statistics |

### Accessing State in Agents

In agent prompts, use:
- `{{ $flow.state.items }}` - Full items array
- `{{ $iteration.current }}` - Current item (Processor only)
- `{{ $iteration.index }}` - Current iteration index (0-based)
- `{{ $iteration.results }}` - All results (Aggregator only)

---

## Troubleshooting

### Error: "Iteration Node not found"

**Cause**: Flowise version too old (< 1.8.0)

**Solution**:
1. Check Flowise version: Settings → About
2. If < 1.8.0, upgrade:
   ```bash
   npm update -g flowise
   # or
   docker pull flowiseai/flowise:latest
   ```

### Error: "Tool 'currentDateTime' not found"

**Cause**: Custom tool not created before import

**Solution**:
1. Go to Tools → Custom Tools
2. Create `currentDateTime` tool (see Prerequisites section)
3. Re-import workflow

### Error: "Agent not connected"

**Cause**: Edge connection missing during import

**Solution**:
1. Manually connect nodes:
   - Drag from output anchor (right side of node)
   - Drop on input anchor (left side of target node)
2. Follow connection order: Start → Planner → Iteration → Processor → Aggregator → Direct Reply

### Warning: "Flow State variable not resolving"

**Cause**: Incorrect variable syntax

**Solution**:
- ✅ Correct: `{{ $flow.state.items }}`
- ❌ Wrong: `$flow.items` or `{{ flow.state.items }}`

### Issue: Processor executes only once

**Cause**: Processor not connected to Iteration Node output

**Solution**:
1. Verify edge exists: `iterationAgentflow_0-output` → `agent_processor-input`
2. Check Iteration Node input: Should show `{{ $flow.state.items }}`
3. Re-import workflow if connections corrupt

---

## Advanced Customization

### Add Progress Tracking

Modify Processor Agent to update state after each iteration:

```javascript
// In Processor system prompt, add:
After processing, update Flow State:
$flow.state.processed_count = {{ $iteration.index }} + 1
```

### Add Timeout Handling

Modify Aggregator to detect slow iterations:

```javascript
// Check if any iteration took > 10 seconds
if (iteration_results.some(r => r.duration > 10000)) {
  summary.warnings = ["Some items took longer than expected"];
}
```

### Add Retry Logic

For failed items, store them in state for retry:

```javascript
// In Aggregator:
$flow.state.failed_items = iteration_results
  .filter(r => !r.success)
  .map(r => r.item)
```

---

## Support

For issues or questions:

1. Check [README.md](./README.md) for usage examples
2. Review [architecture.md](./.context-foundry/architecture.md) for technical details
3. Run validation: `python3 validate_workflow.py 07-batch-processing.json`
4. Check Flowise logs for runtime errors

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2025-01-19 | Initial release with 9-node batch processing pattern |

---

🤖 Built with Context Foundry
