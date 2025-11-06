# Pattern #7: Batch Processing - Testing Guide

**Pattern:** Batch Processing (Iteration Node)
**Version:** 1.0
**Last Updated:** 2025-11-05
**Repository:** https://github.com/snedea/afv2-pattern-07-batch

---

## 🎯 Overview

This guide provides comprehensive test cases for the **Batch Processing** pattern, which demonstrates:
- **Iteration Node** for-each loops over arrays
- Sequential processing of multiple items
- Result aggregation from N iterations
- Graceful handling of edge cases (empty arrays, large datasets)

---

## 🏗️ Pattern Architecture

**Flow:** Start → Planner → Iteration Node → [Processor Agent (N times)] → Aggregator → Direct Reply

**Key Components:**
- **Planner Agent:** Extracts items into array format
- **Iteration Node:** For-each loop (executes Processor once per item)
- **Processor Agent:** Analyzes/transforms each item
- **Aggregator Agent:** Combines results from all iterations
- **Direct Reply:** Terminal node with final output

**Nodes:** 9 total (1 Start, 3 Agents, 1 Iteration Node, 1 Direct Reply, 3 Sticky Notes)
**Edges:** 5 connections

---

## 📋 Test Case Matrix

| Test ID | Scenario | Input Size | Expected Outcome | Duration |
|---------|----------|------------|------------------|----------|
| TC-7.1 | Basic array processing | 3 items | All items processed, aggregated | ~30-45s |
| TC-7.2 | Empty array handling | 0 items | Graceful skip, no errors | ~8-10s |
| TC-7.3 | Large array stress test | 10 items | All items processed sequentially | ~90-120s |

---

## 🧪 Test Case Details

### Test Case TC-7.1: Basic Array Processing

**Objective:** Validate Iteration Node processes array correctly with 3 items

#### Prerequisites
- ✅ Flowise instance running (http://localhost:3000)
- ✅ Pattern imported into Flowise
- ✅ Anthropic API key configured for all 3 agents
- ✅ Model selected: `claude-sonnet-4-5-20250929`

#### Import Steps
1. Open Flowise UI → **Agentflows** → **Add New**
2. Click **"Load Agentflow"**
3. Select `07-batch-processing.json`
4. Verify 9 nodes load correctly:
   - 1 Start node (green)
   - 3 Agent nodes (teal): Planner, Processor, Aggregator
   - 1 Iteration Node (purple): "For Each Loop"
   - 1 Direct Reply node (turquoise)
   - 3 Sticky Notes (yellow)
5. Click **"Save Agentflow"**

#### Configuration
1. **Agent.Planner:**
   - Model: `claude-sonnet-4-5-20250929` ✅ (should be pre-selected)
   - Credential: "Anthropic API Key"
   - Temperature: 0.4

2. **Agent.Processor:**
   - Model: `claude-sonnet-4-5-20250929` ✅ (should be pre-selected)
   - Credential: "Anthropic API Key"
   - Temperature: 0.4

3. **Agent.Aggregator:**
   - Model: `claude-sonnet-4-5-20250929` ✅ (should be pre-selected)
   - Credential: "Anthropic API Key"
   - Temperature: 0.4

#### Test Input
```
Analyze sentiment for these reviews:
1. "This product is amazing! Best purchase ever."
2. "Terrible quality, broke after one day."
3. "It's okay, nothing special but works fine."
```

#### Expected Output

**Planner Output:**
```json
{
  "items": [
    "This product is amazing! Best purchase ever.",
    "Terrible quality, broke after one day.",
    "It's okay, nothing special but works fine."
  ],
  "count": 3
}
```

**Iteration Node:**
- Loop 1: Process item "This product is amazing..."
- Loop 2: Process item "Terrible quality..."
- Loop 3: Process item "It's okay..."

**Processor Output (per iteration):**
- Item 1: Sentiment = **Positive** (score: 0.95)
- Item 2: Sentiment = **Negative** (score: 0.15)
- Item 3: Sentiment = **Neutral** (score: 0.50)

**Aggregator Output:**
```
📊 Sentiment Analysis Report

Total Reviews Processed: 3

Sentiment Breakdown:
✅ Positive: 1 review (33%)
❌ Negative: 1 review (33%)
⚖️ Neutral: 1 review (33%)

Individual Results:
1. "This product is amazing..." → POSITIVE (0.95)
2. "Terrible quality..." → NEGATIVE (0.15)
3. "It's okay..." → NEUTRAL (0.50)

Overall Sentiment: Balanced (mixed opinions)
```

#### Validation Checklist
- [ ] All 3 reviews processed (check Aggregator output mentions "3 reviews")
- [ ] Iteration counter increments correctly: 0 → 1 → 2 → 3
- [ ] Each sentiment classification is correct (Positive, Negative, Neutral)
- [ ] No errors in execution log
- [ ] Processor agent executes exactly 3 times (not 1, not 4)
- [ ] Direct Reply terminal node fires (workflow ends cleanly)
- [ ] Execution time < 60 seconds
- [ ] State variable `batch.results[]` contains 3 items

#### Common Issues & Debugging

**Issue:** Processor runs only once instead of 3 times

**Root Cause:** Iteration Node `iterationInput` not set correctly

**Fix:**
1. Click Iteration Node ("For Each Loop")
2. Check "Array Input" field
3. Should contain: `{{ $flow.state.batch.items }}`
4. Verify Planner agent outputs array in state updates:
   ```json
   {
     "key": "batch.items",
     "value": "{{ items }}"
   }
   ```

---

**Issue:** Error "Cannot iterate over undefined"

**Root Cause:** Planner didn't create array in flow state

**Fix:**
1. Check Planner agent output in execution log
2. Verify it extracted items: `["item1", "item2", "item3"]`
3. Check Planner's `agentStateUpdates` includes:
   - Key: `batch.items`
   - Value: `{{ items }}`

---

### Test Case TC-7.2: Empty Array Handling

**Objective:** Validate graceful handling of empty/no input

#### Test Input
```
Process this list of items: (none provided)
```

#### Expected Behavior
1. **Planner:** Detects no items provided
2. **Iteration Node:** Receives empty array `[]`, skips loop (0 iterations)
3. **Processor:** Never executes
4. **Aggregator:** Receives empty results array, returns:
   ```
   ℹ️ No items to process.

   Please provide a list of items to analyze.
   ```

#### Validation Checklist
- [ ] No errors or exceptions thrown
- [ ] Processor agent executes 0 times (not even once)
- [ ] Aggregator acknowledges empty input gracefully
- [ ] Direct Reply fires with helpful message
- [ ] Execution time < 15 seconds
- [ ] Workflow completes successfully (not stuck)

---

### Test Case TC-7.3: Large Array Stress Test

**Objective:** Validate performance and stability with 10 items

#### Test Input
```
Analyze sentiment for these 10 product reviews:
1. "Great product, highly recommend!"
2. "Poor quality, very disappointed."
3. "Average, meets basic expectations."
4. "Excellent value for money!"
5. "Broke within a week, waste of money."
6. "Good quality, would buy again."
7. "Not worth the price, too expensive."
8. "Perfectly fine, does the job."
9. "Outstanding product, exceeded expectations!"
10. "Terrible experience, do not buy."
```

#### Expected Output
- **Total items processed:** 10
- **Iteration executions:** 10 (Processor runs 10 times)
- **Sentiment breakdown:**
  - Positive: 4 reviews (40%)
  - Negative: 3 reviews (30%)
  - Neutral: 3 reviews (30%)

#### Performance Expectations
- **Execution time:** 90-120 seconds (~10 seconds per item)
- **Memory usage:** Stable (no leaks)
- **No timeouts:** All 10 iterations complete

#### Validation Checklist
- [ ] All 10 items appear in Aggregator output
- [ ] Iteration counter reaches 10
- [ ] No "timeout" or "max iterations" errors
- [ ] Results array has exactly 10 entries
- [ ] Execution completes within 150 seconds
- [ ] Flowise UI remains responsive

---

## 🔍 Advanced Testing Scenarios

### Scenario A: Mixed Data Types in Array

**Test Input:**
```
Process these items:
1. "Review: Great product"
2. 42
3. {"type": "review", "text": "Good"}
```

**Expected:** Processor handles different data types, or returns error with helpful message

---

### Scenario B: Very Large Array (50+ items)

**Warning:** This may take 8-10 minutes

**Test Input:** 50 short reviews

**Expected:**
- All 50 items processed
- No memory issues
- Progress visible in execution log

---

### Scenario C: Interruption During Iteration

**Objective:** Test workflow resilience

**Steps:**
1. Start processing 10 items
2. After 3rd iteration, stop workflow manually
3. Restart workflow with same input

**Expected:**
- Workflow restarts from beginning (iterations aren't cached)
- No corrupted state

---

## 📊 Test Execution Report Template

Use this template to document your test results:

```markdown
## Pattern #7 Test Execution Report

**Date:** 2025-11-05
**Tester:** [Your Name]
**Flowise Version:** [e.g., v2.1.0]
**Environment:** [Local/Cloud]

### Test Results

| Test Case | Status | Duration | Items Processed | Notes |
|-----------|--------|----------|-----------------|-------|
| TC-7.1: Basic (3 items) | ✅ PASS | 38s | 3/3 | All sentiments correct |
| TC-7.2: Empty array | ✅ PASS | 9s | 0/0 | Graceful handling |
| TC-7.3: Large (10 items) | ✅ PASS | 105s | 10/10 | Completed without errors |

### Issues Found
- None

### Performance Metrics
- Average time per item: ~10-12 seconds
- Memory usage: Stable
- CPU usage: Moderate

### Overall Assessment
✅ **PASS** - Pattern ready for production use

### Recommendations
- For arrays > 20 items, consider adding progress indicator
- Consider adding batch size limit (e.g., max 100 items)
```

---

## 🎯 Success Criteria

Mark testing complete when:
- [ ] TC-7.1 (Basic array) passes
- [ ] TC-7.2 (Empty array) passes
- [ ] TC-7.3 (Large array) passes
- [ ] No infinite loops or crashes
- [ ] Iteration Node loops correct number of times
- [ ] Aggregator properly combines results
- [ ] Direct Reply terminal works correctly

---

## 🐛 Known Issues

None currently. Report issues at: https://github.com/snedea/afv2-pattern-07-batch/issues

---

## 📚 Additional Resources

- **Pattern README:** `README.md` in this repository
- **Integration Guide:** `INTEGRATION_GUIDE.md`
- **Full Test Suite:** [Context Foundry TESTING_GUIDE.md](https://github.com/context-foundry/context-foundry/blob/main/extensions/flowise/templates/afv2-patterns/TESTING_GUIDE.md)
- **Flowise Documentation:** https://docs.flowiseai.com/using-flowise/agentflowv2

---

**Version History:**
- **1.0** (2025-11-05): Initial test suite with 3 test cases
