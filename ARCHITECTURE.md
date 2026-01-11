# EVE Architecture

## Overview

EVE is built on a modular architecture that separates concerns into distinct layers:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         EVE PLATFORM                                 │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                     GENESIS ENGINE                           │   │
│  │  "Natural language → Production code"                        │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│          ┌───────────────────┼───────────────────┐                 │
│          ▼                   ▼                   ▼                 │
│  ┌───────────────┐   ┌───────────────┐   ┌───────────────┐        │
│  │ Code Factory  │   │  Claude API   │   │  Local LLM    │        │
│  │ (Templates)   │   │  (Premium)    │   │  (Qwen/Llama) │        │
│  └───────────────┘   └───────────────┘   └───────────────┘        │
│          │                   │                   │                 │
│          └───────────────────┼───────────────────┘                 │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    VALIDATION LAYER                          │   │
│  │  AST Check → Syntax → Import → Unit Tests → Integration     │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                              │                                      │
│                              ▼                                      │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                    KNOWLEDGE LAYER                           │   │
│  │  Semantic RAG │ Domain Knowledge │ Feedback Loop            │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Genesis Engine

The orchestrator that transforms natural language into code.

**Pipeline:**
```
Input: "Build a BatteryMonitorEngine with SoC calculation"
  │
  ├─→ Task Decomposition
  │     └─→ type: engine, name: BatteryMonitor
  │
  ├─→ Plan Generation
  │     └─→ files: [main.py, test.py, __init__.py]
  │
  ├─→ Smart Routing (decides: CodeFactory | Claude | LocalLLM)
  │
  ├─→ Code Generation
  │
  ├─→ Validation & AutoFix
  │
  └─→ Output: Tested, working code
```

### 2. Smart Router

Intelligent routing based on task complexity:

| Signal | Route To |
|--------|----------|
| Dashboard, widget, API keywords | Code Factory |
| Security, encryption, auth + JWT | Claude API |
| Complex + multiple indicators | Claude API |
| Standard engine/module | Local LLM |

**Routing Logic (Conceptual):**
```python
def route(task):
    if is_deterministic_pattern(task):
        return CODE_FACTORY      # 2ms, free
    
    if complexity_score(task) >= THRESHOLD:
        return CLAUDE_API        # 30s, premium
    
    return LOCAL_LLM             # 2-4min, free
```

### 3. Code Factory

Template-based generation for predictable patterns:

- **Input:** Structured spec (widgets, API endpoints, etc.)
- **Output:** Complete project with all files
- **Speed:** 1-5ms
- **Accuracy:** 100% (deterministic)

**Supported Patterns:**
- React dashboards with widgets
- FastAPI backends
- CRUD APIs
- Standard data models

### 4. Semantic RAG

GPU-accelerated retrieval of relevant code examples:

```
Query: "battery monitor energy"
  │
  ├─→ Embed query (sentence transformer)
  │
  ├─→ Search 700+ files (CUDA cosine similarity)
  │
  └─→ Return top 5:
        • battery_engine.py (92% match)
        • energy_safety_agent.py (78% match)
        • solar_panel_engine.py (71% match)
        • ...
```

**Key Features:**
- Domain-agnostic (no hardcoded keywords)
- Self-improving (new code is indexed)
- GPU-accelerated for speed

### 5. AutoFix Loop

Automatic error detection and correction:

```
Code Generated
  │
  ├─→ AST Parse (syntax check)
  │     └─→ Error? → Fix attempt 1
  │
  ├─→ Import Check
  │     └─→ Missing? → Add imports
  │
  ├─→ Unit Tests
  │     └─→ Fail? → Fix attempt 2
  │
  └─→ Max 5 iterations → Success or report
```

### 6. Feedback Loop (Learning)

Every successful build improves future generations:

```
Successful Build
  │
  ├─→ Save to knowledge base
  │
  ├─→ Index in semantic search
  │
  └─→ Future builds find this code as example
```

## Data Flow

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  User    │───▶│  Parse   │───▶│  Route   │───▶│ Generate │
│  Input   │    │  Intent  │    │  Smart   │    │   Code   │
└──────────┘    └──────────┘    └──────────┘    └────┬─────┘
                                                     │
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌────▼─────┐
│  Output  │◀───│  Learn   │◀───│  Test    │◀───│ Validate │
│  Code    │    │  Index   │    │  Fix     │    │   AST    │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

## Technology Choices

### Why Hybrid AI Routing?

| Approach | Pros | Cons |
|----------|------|------|
| Always LLM | Flexible | Slow, expensive, unpredictable |
| Always Templates | Fast, free | Limited flexibility |
| **Hybrid (EVE)** | **Best of both** | More complex to build |

### Why Local LLM Option?

1. **Privacy:** Code never leaves your machine
2. **Cost:** Zero API costs for routine tasks
3. **Offline:** Works without internet
4. **Compliance:** Required for regulated industries

### Why Semantic Search (not keyword)?

```
Keyword search: "battery" → finds files with "battery" in name
Semantic search: "energy storage" → finds battery_engine.py

Semantic understands MEANING, not just words.
```

## Security Considerations

- API keys stored locally (never in code)
- Local LLM option for sensitive projects
- No telemetry or data collection
- All processing can be air-gapped

## Scalability

| Component | Scaling Strategy |
|-----------|-----------------|
| Code Factory | Stateless, instant |
| Local LLM | GPU memory bound |
| Claude API | API rate limits |
| RAG Search | GPU parallelization |

## Future Architecture

Planned additions:
- Distributed code generation
- Multi-agent collaboration
- Real-time code streaming
- IDE integrations (VS Code, JetBrains)

---

*This document describes the conceptual architecture. Implementation details are proprietary.*
