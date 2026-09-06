# A Stratified Cognitive Memory System
## Based on Attractor–Constraint Dynamics

---

## 1. CORE PRINCIPLE

The system is based on a single engineering principle:

**Cognitive stability emerges from the controlled interaction between attractors (driving forces) and constraints (limiting rules).**

This is not treated as philosophy, but as a design invariant:

- **Attractors** = scoring, selection, prioritization functions
- **Constraints** = hard/soft invariants and validation rules
- **Stability** = bounded information flow across layers

---

## 2. SYSTEM OVERVIEW

The memory system is composed of four strictly separated layers:

- **L1 — Immutable Event Store** (ground-truth trace)
- **L2 — User Context Memory** (persistent relevance store)
- **L3 — Epistemic Evaluation Layer** (probabilistic validation layer)
- **L4 — Working Memory / Attention Layer** (active reasoning space)

### Core invariant

**Information may move upward in abstraction, but never downward in authority.**

---

## 3. CORE DATA MODEL

### 3.1 Canonical Memory Item

```python
class MemoryItem:
    id: str
    timestamp: float
    
    content: dict
    source: str
    
    embedding: list[float] | None = None
    tags: set[str] = set()
    
    confidence: float = 0.0
    metadata: dict = {}
```

---

## 4. LAYER 1 — IMMUTABLE EVENT STORE

### 4.1 Role

Provides an append-only, fully traceable historical log.

### 4.2 Structure

```python
class Event(MemoryItem):
    hash: str
    prev_hash: str | None
    immutable: bool = True
```

### 4.3 Storage

```python
Layer1 = list[Event]  # append-only log
```

### 4.4 Rules

- Append-only
- No updates
- No deletions
- No reinterpretation of stored data

### 4.5 Write operation

```python
def write_L1(event: Event):
    event.hash = hash((event.content, event.timestamp))
    Layer1.append(event)
```

---

## 5. LAYER 2 — USER CONTEXT MEMORY

### 5.1 Role

Stores long-term, user-relevant information with decay-based persistence.

### 5.2 Structure

```python
class ContextItem(MemoryItem):
    relevance: float
    last_accessed: float
    decay_rate: float
    
    user_editable: bool = True
```

### 5.3 Storage

```python
Layer2 = dict[str, ContextItem]
```

### 5.4 Relevance update

```python
def update_relevance(item, context_vector):
    similarity = cosine_similarity(item.embedding, context_vector)
    time_decay = exp(-(now() - item.last_accessed))
    
    item.relevance = similarity * time_decay
```

### 5.5 Retention rule

```python
def should_keep(item):
    return item.relevance > RELEVANCE_THRESHOLD
```

---

## 6. LAYER 3 — EPISTEMIC EVALUATION LAYER

### 6.1 Role

Performs probabilistic classification and consistency evaluation.

**This is NOT a truth engine, but a scoring system.**

### 6.2 Structure

```python
class EpistemicState:
    item_id: str
    
    label: str  # fact | hypothesis | speculation | fiction
    
    truth_score: float
    consistency_score: float
    
    evidence_strength: float
    contradiction_score: float
```

### 6.3 Evaluation pipeline

```python
def evaluate(item):
    evidence = retrieve_evidence_from_L1(item)
    contradictions = detect_contradictions(item, Layer1)
    
    return EpistemicState(
        item_id=item.id,
        label=classify(item),
        truth_score=compute_truth(evidence, contradictions),
        consistency_score=compute_consistency(item),
        evidence_strength=len(evidence),
        contradiction_score=len(contradictions)
    )
```

### 6.4 Probabilistic truth scoring

```python
def compute_truth(evidence, contradictions):
    return sigmoid(len(evidence) - 2 * len(contradictions))
```

### 6.5 Epistemic gate (soft constraint)

```python
def epistemic_gate(state):
    return (
        state.truth_score >= MIN_TRUTH_THRESHOLD
        or state.label in ["hypothesis", "speculation"]
    )
```

👉 **Nothing is strictly forbidden; everything is weighted and classified.**

---

## 7. LAYER 4 — WORKING MEMORY / ATTENTION LAYER

### 7.1 Role

Short-lived workspace for active reasoning and response generation.

### 7.2 Structure

```python
class WorkingMemory:
    active_items: list[MemoryItem]
    
    attention_weights: dict[str, float]
    task_context: dict
    
    ttl: float
```

### 7.3 Activation function

```python
def activate(item, task_context):
    score = (
        alpha * recency(item)
        + beta * relevance(item, task_context)
        + gamma * goal_alignment(item)
    )
    
    WorkingMemory.attention_weights[item.id] = score
```

### 7.4 Focus selection

```python
def select_focus():
    return max(
        WorkingMemory.active_items,
        key=lambda x: WorkingMemory.attention_weights[x.id]
    )
```

### 7.5 Expiration rule

```python
def cleanup():
    WorkingMemory.active_items = [
        item for item in WorkingMemory.active_items
        if now() - item.timestamp < WorkingMemory.ttl
    ]
```

---

## 8. ORCHESTRATION LAYER (CORE LOOP)

### 8.1 Cognitive cycle

```python
def cognitive_step(input_event, task_context):
    
    # 1. Ingestion
    event = parse(input_event)
    write_L1(event)
    
    # 2. Epistemic evaluation
    state = evaluate(event)
    
    # 3. Context update (conditional persistence)
    if state.truth_score > CONTEXT_THRESHOLD:
        update_L2(event)
    
    # 4. Attention activation
    activate(event, task_context)
    
    # 5. Focus selection
    focus = select_focus()
    
    # 6. Generate response
    return generate_response(focus)
```

---

## 9. CROSS-LAYER CONSTRAINTS

### 9.1 Allowed information flow

```
L1 → L3 → L2 → L4
```

### 9.2 Forbidden flows

- L4 → L2 (attention cannot become memory)
- L2 → L1 (context cannot rewrite history)
- L3 → L1 (interpretation cannot modify events)

### 9.3 Implementation interpretation

Constraints are enforced through:

- access control rules
- scoring thresholds
- immutability guarantees
- pipeline ordering

**not metaphysical absolutes.**

---

## 10. SYSTEM STABILITY MODEL

### 10.1 Stability function

```python
stability = balance(
    epistemic_signal(L3),
    memory_entropy(L2),
    attention_variance(L4)
)
```

### 10.2 Failure modes

| Mode | Cause |
|------|-------|
| hallucination drift | weak epistemic filtering (L3) |
| memory pollution | over-permissive persistence (L2) |
| attention noise | unstable weighting (L4) |
| rigidity | overly strict filtering |

---

## 11. EMERGENT SYSTEM PROPERTIES

When correctly tuned, the system exhibits:

✔ **Traceability**
- Full reconstruction via L1 event log

✔ **Personalization**
- Adaptive relevance via L2

✔ **Epistemic filtering**
- Probabilistic validation via L3

✔ **Attention control**
- Dynamic prioritization via L4

---

## 12. SYSTEM CLASSIFICATION

This architecture is:

**A stratified memory-augmented agent system with probabilistic epistemic evaluation and constrained attention dynamics.**

It is NOT:

- a truth engine
- a hallucination-proof system
- a deterministic cognitive model

---

## 13. KEY DESIGN STRENGTH

The core innovation is not any single layer, but:

**explicit separation between historical trace, user memory, epistemic evaluation, and active attention under strict directional constraints.**

---

## 14. LIMITATIONS

- truth remains probabilistic, not absolute
- performance depends heavily on embeddings + retrieval quality
- thresholds require empirical tuning
- hallucination risk is reduced, not eliminated

---

## 15. SUMMARY

This system is a structured alternative to standard RAG:

- more explicit memory stratification
- stronger control over persistence
- explicit epistemic scoring layer
- isolated attention workspace

---

**Status:** Engineering framework | **Date:** 2026