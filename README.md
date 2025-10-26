# Reducing LLM Hallucinations in Voice AI Systems

Minimizing hallucinations in conversational AI agents, with specific focus on SenseHQ's Voice AI

---

## Table of Contents
- Overview
- Core Mitigation Strategies
  - Retrieval-Augmented Generation
  - Structured Prompt Engineering
  - Post-Generation Verification
  - Temperature and Decoding Control
  - Domain Fine-Tuning
- Performance Comparison


---

## Overview

Large Language Models generate plausible but factually incorrect information (hallucinations), creating critical risks in voice AI systems where users expect accurate, real-time responses.

## Core Mitigation Strategies

### 1. Retrieval-Augmented Generation (RAG)

**Purpose**: Ground model responses in verified external data sources before generation.

**How It Works**:
- Query vector databases before LLM generation
- Retrieve top 3-5 relevant documents from verified knowledge base
- Inject retrieved context directly into the prompt
- Instruct model to answer only from provided context

**Voice AI Application Example**:

Retrieved context from knowledge base:
- SenseHQ transcription latency: 50ms average
- Supported languages: English, Spanish
- Call recording retention: 90 days

System instruction: "Answer using only provided context. State explicitly if information is unavailable."

User query: "What's your transcription speed?"

Agent response: "Our average transcription latency is 50 milliseconds for real-time processing."

**Impact**: reduction in factual errors

**Trade-offs**:
- Infrastructure overhead: Vector database maintenance, embedding pipeline
- Latency addition: 20-100ms per retrieval operation
- Token cost increase: 200-500 additional tokens per query

**Recommended For**:
- Call analytics queries
- Dynamic information (pricing, availability, schedules)

---

### 2. Structured Prompt Engineering

**Purpose**: Constrain model output through explicit instructions, format specifications, and uncertainty handling protocols.

**Core Techniques**:

#### Off-Topic Guardrails
Define strict conversation boundaries to prevent scope drift. For recruiting agents, explicitly prohibit discussions about protected categories: race, color, religion, sex, national origin, age, disability status, genetic information.

Response pattern: "I'm here to discuss the job opportunity. Let's focus on your qualifications."

#### Uncertainty Handling
Implement explicit uncertainty protocols:
- Answer only with available information
- State "I don't have that information" when uncertain
- Never invent facts about policies, dates, people, or processes
- Offer human escalation when appropriate

#### Format Constraints
Enforce structured outputs to prevent prose-based hallucinations:
- JSON-only responses for data extraction
- Bullet point limits (3 bullets, max 20 words each)
- Required fields with validation
- No explanatory text outside defined structure

#### Voice-Specific Normalization
**Pronunciation Rules**:
- "401K" → "Four-O-One-K" (never "four hundred one K")
- URLs: "nklaundry.com" → "en kay laundry dot com"
- Currency: "$1850" → "eighteen hundred fifty dollars" (never "eighteen fifty")
- Ranges: "1-2 years" → "one to two years of experience"
- Special characters: Remove underscores, hyphens, slashes from job titles

**Roman Numeral Conversion**:
- "V" → "Five"
- "XII" → "Twelve"
- Apply to all job titles and descriptions before vocalization

**Impact**: reduction in unsupported claims

**Trade-offs**:
- Reduced conversational flexibility
- Longer prompt token counts (mitigated through optimization)
- Requires comprehensive rule definition

---

### 3. Post-Generation Verification

**Purpose**: Cross-validate outputs through self-consistency checks or secondary model verification.

**Methods**:

#### Self-Consistency Approach
Generate 3-5 candidate responses with different random seeds. Compare outputs for agreement. Accept consensus answers present in majority of responses. Flag contradictions for human review or request user clarification.

#### Critic Model Pattern
Use secondary LLM in verification mode to review initial output. Compare against source context for factual accuracy. Identify unsupported claims or logical inconsistencies. Return confidence-scored final response.

**Voice AI Workflow**:

Step 1: Generate transcription with primary ASR model

Step 2: Generate parallel transcription with secondary model

Step 3: Compare results
- If agreement ≥90%: Accept consensus transcription
- If disagreement >10%: Request user confirmation - "I want to make sure I have this right. Did you say [consensus text]?"

Step 4: If persistent disagreement: Flag for human review

**Impact**: reduction through consistency filtering

**Trade-offs**:
- Latency increase
- Infrastructure complexity: Multi-model orchestration

**Use Cases**:
- Critical compliance transcription (legal, medical, financial)
- Multi-party call summarization
- Transaction verification
- High-stakes customer interactions

---

### 4. Temperature and Decoding Control

**Purpose**: Reduce generation randomness for deterministic, factual outputs.

**Configuration Strategy**:

Set temperature to 0.0 for maximum determinism. Use top_p value of 1.0 to disable nucleus sampling. Apply greedy decoding for factual tasks. Eliminate randomness in token selection process.

**Task-Specific Settings**:

| Task Type | Temperature | Top-p | Reasoning |
|-----------|-------------|-------|-----------|
| Call transcription | 0.0 | 1.0 | Maximum accuracy required, no creativity needed |
| Metadata extraction | 0.0 | 1.0 | Structured data extraction, deterministic output |
| Compliance responses | 0.0 | 1.0 | Zero hallucination tolerance |
| Appointment scheduling | 0.2 | 0.95 | Slight variation for natural phrasing |
| General conversation | 0.5-0.7 | 0.9 | Natural dialogue flow required |
| Empathetic responses | 0.7 | 0.9 | Emotional nuance and variation valued |

**Impact**: improvement in factual accuracy

**Trade-offs**:
- Less varied outputs (repetitive phrasing possible)
- More predictable responses (acceptable for factual tasks)
- Reduced conversational dynamism

**Recommended For**:
- Data extraction tasks
- Compliance documentation
- Factual query responses
- Structured information retrieval

---

### 5. Domain Fine-Tuning

**Purpose**: Align model's internal knowledge representation with verified domain-specific data.

**Training Data Structure**:

Curate instruction-context-response triples from verified sources:
- Instructions: Common user queries in natural language
- Context: Source documentation (technical specs, compliance manuals)
- Responses: Verified, accurate answers

Example training sample:
- Instruction: "What is SenseHQ's average transcription latency?"
- Context: "Technical documentation v4.2"
- Response: "SenseHQ achieves 50ms processing latency for real-time transcription using optimized ASR models."

**Fine-Tuning Approach**:

Use parameter-efficient methods (LoRA, QLoRA) to reduce training costs. Target 500-2000 high-quality training examples per domain. Combine with RAG for dynamic information that changes frequently. Retrain monthly or when major product updates occur.

**Data Curation Requirements**:
- Verify all responses against authoritative sources
- Include edge cases and common misunderstandings
- Cover full scope of expected user queries

**Impact**: reduction in domain-specific hallucinations

**Trade-offs**:
- High curation cost
- Maintenance burden for knowledge updates
- Model versioning and deployment complexity

**Recommended For**:
- Specialized terminology (medical, legal, technical)
- Company-specific processes and policies
- Industry-specific compliance requirements

---

## Performance Comparison

| Strategy | Hallucination Reduction | Latency Impact | Token Usage Increase | Implementation Complexity | Best For |
|----------|------------------------|----------------|-------------------|---------------------------|----------|
| RAG | Very High | Low-Medium | Medium | High | Factual queries, dynamic data |
| Structured Prompts | Medium-High | Minimal | Variable | Medium | Guardrails, format enforcement |
| Verification Loops | High | Medium-High | High | Medium | Critical accuracy (compliance, legal) |
| Temperature=0 | Medium | None | None | Low | Deterministic tasks (transcription, extraction) |
| Fine-Tuning | High | None | None | Very High | Domain-specific terminology |

**Combined Strategy Impact**: 
- Implementing multiple techniques yields cumulative benefits. RAG + Structured Prompts + Temperature=0 can achieve a huge reduction in hallucination for factual tasks.
---

## Common Pitfalls and Solutions

### Pitfall 1: Over-Optimization Leading to Unnatural Responses
**Problem**: Extreme token reduction creates robotic, terse responses.

**Solution**: Maintain conversational markers for voice interactions. Keep pronouns, natural transitions. Optimize system instructions heavily, but preserve conversational flow in response generation.

---

### Pitfall 2: RAG Retrieval Returning Irrelevant Context
**Problem**: Injected context doesn't match user query, model hallucinates connections.

**Solution**: Implement relevance scoring threshold. Only inject documents with similarity score >0.75. Include explicit instruction: "If provided context doesn't answer the query, state this explicitly."

---

### Pitfall 3: Fine-Tuned Models Become Stale
**Problem**: Model knowledge drifts from current product state after deployment.

**Solution**: Establish monthly retraining schedule. Combine fine-tuning with RAG for dynamic information. Version control training datasets. Maintain changelog for knowledge base updates.

---

### Pitfall 4: Timezone Calculations Introduce Errors
**Problem**: Daylight saving time, timezone abbreviation ambiguities cause scheduling hallucinations.

**Solution**: Always use IANA canonical timezone names (America/Los_Angeles, not PST). Explicitly confirm timezone with user for all specific time requests. Validate all calculated times are future times within 7-day window before accepting.




