# Deep Technical Overview: On-Device Generative AI Architecture

This document provides an exhaustive technical breakdown of the AI systems powering **RVX AI NOTES**. The application runs a **local Llama 3.2 1B language model** directly on the user's device via **Meta's ExecuTorch** inference runtime — eliminating cloud calls entirely.

---

## 1. AI Architecture Overview

The app uses a **three-tier intelligence strategy**:

| Tier | Engine | Latency | Use Case |
|---|---|---|---|
| **Tier 1: Heuristics** | Rule-based structural parsing | ~0ms | Topic boundaries (headers, lists, dividers) |
| **Tier 2: Statistical NLP** | BM25 + LexRank + TextRank + LSA + MMR | ~300–800ms | Extractive summaries & fallback |
| **Tier 3: On-Device LLM** | Llama 3.2 1B SpinQuant via ExecuTorch | ~60–120s | Abstractive summaries & whole-note analysis |

> **Note:** The cloud LLM (Gemini API) integration is an optional Tier 4 — not the primary AI path. The default experience is fully offline.

---

## 2. On-Device LLM Engine (`lib/llmSummarizer.ts`)

### Model Details
| Property | Value |
|---|---|
| **Model** | Llama 3.2 1B SpinQuant |
| **Format** | `.pte` (ExecuTorch portable format) |
| **Size** | ~500–600MB |
| **Runtime** | `react-native-executorch ^0.8.3` |
| **Download** | Explicit user-initiated from Settings |
| **Storage** | Device-local cache via `ExpoResourceFetcher` |
| **Load time** | ~3–8s into native memory |
| **KV cache** | ~50–100MB at inference time |

### Initialization Sequence
```
App Launch
  └── initLLM() [non-blocking, background]
        └── isModelDownloaded()
              ├── YES → _loadModelIntoMemory()
              │         └── LLMModule.fromModelName(LLAMA3_2_1B_SPINQUANT)
              │               → _llmInstance ready
              └── NO  → wait for user-initiated download
```

### Inference Configuration
```typescript
llm.configure({
    generationConfig: {
        temperature: 0.3,   // Low temp → structured, factual output
        topp: 0.9,          // Nucleus sampling
    },
});
```

### Content-Type Aware Prompting (Feynman Variants)
The `buildSummaryPrompt()` function detects content type and selects a matching Feynman-style prompt template:

| `ContentType` | Prompt Strategy | Output Format |
|---|---|---|
| `theory` / `mixed` | "Explain like a teacher" → analogy + mechanism | Analogy · Mechanism · Exam Tip · Recall cue |
| `command-reference` | "Create a command cheat sheet" | Command list with purpose |
| `tutorial` | "Summarize these steps" | Numbered step condensation |
| `reference` | "Explain this code" | Code explanation with context |

### State Management & Listeners
The engine uses a singleton pattern with event emitters for UI status updates:

```typescript
interface LLMStatus {
    isReady: boolean;
    isGenerating: boolean;
    downloadProgress: number;  // 0–1
    error: string | null;
    modelLoaded: boolean;
}

// Subscribe from any component
const unsub = addLLMStatusListener((status: LLMStatus) => { ... });
```

---

## 3. Whole-Note Summarizer (`lib/intelligentSummarizer.ts`)

Generates ONE comprehensive summary for an entire note — displayed in the Summaries tab.

### 7-Stage Pipeline

```
Stage 1: Goal Extraction
  └── detectNoteType(content) → 'lecture' | 'code-study' | 'concept-map' | 'procedure' | 'mixed'
        Rules: code ratio, shell commands, step markers, definition phrases, H2 density

Stage 2: Content Analysis
  └── profileContent(content) → ContentProfile
        Outputs: hasCode, hasDefinitions, hasProcedures, hasFormulas, estimatedComplexity, wordCount

Stage 3: Block Typing
  └── classify content blocks into theory / code / procedural

Stage 4: Architecture Selection
  └── buildPrompt(noteType, profile) → system prompt
        Selects output schema matching note type + content profile

Stage 5: Compression
  └── smartCompressContent(content, 3500) → truncated string
        Priority: headings > definitions > first sentences > examples
        Hard cap at 3500 chars to fit model context window

Stage 6: LLM Generation
  └── llm.generate(prompt, { maxNewTokens: 512 }) → raw output
        Single call — 60–120s generation time

Stage 7: Assembly
  └── Validate, strip artifacts, ensure headings present
  └── Return SummaryGenerationResult { content, source, noteType, keywords }
```

### Output Schema (per note type)

**Lecture/Mixed:**
```
📌 Overview
🧠 Key Concepts
🔑 Key Takeaways
⚡ Quick Recall
```

**Code-Study:**
```
📌 What This Covers
🔧 Core Commands / APIs
💡 How It Works
⚠️ Watch Out For
```

**Procedure:**
```
📌 Goal
📋 Steps (numbered)
⚠️ Prerequisites
🔗 Expected Outcome
```

---

## 4. Note Summary Cache (`lib/noteSummaryCache.ts`)

Summaries are persisted between sessions using a **content-hash keyed file cache**.

- **Location**: `documentDirectory/ai_note_summaries_v1/`
- **Key**: `{noteId}__{djb2Hash(content)}.json`
- **Invalidation**: Hash changes if note content changes → old cache auto-cleared
- **Event emitter**: `onNoteSummaryUpdated(noteId)` notifies UI reactively

```typescript
interface NoteSummary {
    content: string;       // Structured markdown for display
    generatedAt: string;   // ISO timestamp
    source: 'llm' | 'extractive';
    contentHash: string;   // DJB2 hash of original note content
}
```

---

## 5. Smart Topic Parser (`lib/smartTopicParser.ts`)

The parser is the "Orchestrator" — it decides how to split raw notes into revision blocks.

### Multi-Signal Splitting (v2.2)

#### Hard Boundaries (instant, no blank line required)
```
• Markdown H1/H2/H3     →  # heading
• Numbered sections     →  "4. Deployment", "10. Persistent Volume"
• Bold-only lines       →  **Heading** or __Heading__
• Colon headers         →  "Worker nodes:", "API Server:" (≤6 words)
• ALL CAPS headings     →  "TIME COMPLEXITY" (≤8 words)
• Triple dividers       →  --- or === (2+ consecutive)
• Roman numerals        →  "I. Introduction"
```

#### Semantic Shift Detection
```
Jaccard Similarity:
  words_A ∩ words_B
  ─────────────────  < 0.10 → split (topic shifted)
  words_A ∪ words_B

Applied over: current block accumulated context vs. next paragraph
```

#### Tech Entity Dictionary
30+ tech topics (Docker, Kubernetes, Terraform, AWS, Azure, SQL, Python, Git, etc.):
- Each topic has **primary identifiers** (unambiguous tool names)
- And **secondary context words** (supporting terms)
- **Primary-word match** in adjacent blocks triggers hard subject boundary

#### Smart Merge Logic
```
Fragment (<6 words) → always merge into previous
Short block (<20 words) AND prev block not structural-boundary → merge
Different detected tech topics → never merge (subject-aware)
```

---

## 6. Extractive Summarizer (`lib/localSummarizer.ts`)

Custom hybrid pipeline — runs entirely in TypeScript, zero native modules.

### Fused Scoring Formula

$$S = 0.30 \cdot LexRank + 0.22 \cdot BM25 + 0.18 \cdot TextRank + 0.18 \cdot LSA + 0.12 \cdot Structural$$

| Algorithm | Mechanism |
|---|---|
| **LexRank** | TF-IDF cosine similarity matrix → Power Iteration → eigenvector centrality |
| **BM25** | `K1=1.5, B=0.75` — keyword frequency with document-length normalization |
| **TextRank** | PageRank on 3-token sliding window co-occurrence graph |
| **LSA** | SVD (`topics=4`) + Power Iteration → sentence-topic alignment scores |
| **Structural** | Position bonus (lead/conclusion), definition detection, heading proximity |

### Maximal Marginal Relevance (MMR)
$$\text{MMR}(s) = \lambda \cdot S(s) - (1-\lambda) \cdot \max_{s_j \in Selected} \cos(s, s_j)$$

- `λ = 0.65` — balances relevance vs. diversity
- Prevents redundant sentences from appearing in output

### Code Block Scoring
Not all code blocks appear in summaries. Each block is scored and must exceed a threshold:

```
CODE_IMPORTANCE_THRESHOLD = 0.30

Score boosters:
  • Shell/Bash/Terminal commands   → +0.4
  • Syscalls, SQL, imports         → +0.3
  • Short concise snippets         → preferred
  • Long boilerplate               → penalized

COMMAND_LINE_LIMIT = 4 lines → rendered inline (not as fenced block)
```

---

## 7. Deep Summarizer (`lib/deepSummarizer.ts`)

Powers the extractive fallback for block-level summaries. Generates teacher-style explanations without the LLM.

### Explanatory Flow (v2)
Routes sentences through role classification before assembly:

```
Sentence Roles:
  definition   → "X is a...", "X refers to...", "X means..."
  mechanism    → "works by", "operates via", "implements"
  purpose      → "used for", "allows", "enables", "designed to"
  property     → "always", "never", "cannot", "must"
  consequence  → "results in", "causes", "leads to"
  example      → "for example", "e.g.", "such as"

Assembly Order:
  definition → mechanism → purpose → property → consequence → example

Role-matched connectives prevent abrupt jumps between sentences.
```

### Replaced v1 Methods

| v1 Method | Problem | v2 Replacement |
|---|---|---|
| `generateOverview()` | Filled keywords into a fixed template | Finds real definition sentence from notes, uses it as opening line |
| `generateSectionSummary()` | Keyword density → concatenation (no connection) | Routes through `buildExplanatoryFlow()` |
| `generateCodeExplanation()` | One-line label "The following code defines X" | Reads code AST-lite: extracts function signatures, YAML fields, commands, loops |
| `generateKeyTakeaways()` | First sentences + "Core concepts: X, Y" | Targets exam content specifically: definitions, distinctions, when-to-use |

---

## 8. Spaced Repetition Engine (SM-2)

Every topic block is scheduled using a custom SM-2 implementation in `NotesContext`.

### Variables per Topic
```typescript
interface TopicProgress {
    interval: number;      // Days until next revision
    easeFactor: number;    // Complexity multiplier (default: 2.5)
    dueDate: string;       // ISO date string
    reviewCount: number;   // Successful revisions so far
    lastRating: 'easy' | 'good' | 'hard' | 'again' | null;
}
```

### Update Formula
$$EF' = \max(1.3, EF + (0.1 - (5 - q)(0.08 + (5 - q) \cdot 0.02)))$$

### Interval Progression
| Review | Interval |
|---|---|
| 1st | 1 day |
| 2nd | 3 days |
| n-th | `interval × EF` |
| Rating: Again | Reset to 1 day |

### Feed Interleaving Algorithm
```
Priority weights:
  'again' rated topics → 3× appearance frequency

Interleaving:
  1 "Hard/Again" topic : 2 "Regular/Due" topics
  → prevents overwhelm while keeping difficult content visible
```

---

## 9. Data Persistence & Integrity

### Dual-Storage Strategy
| Store | Engine | Data |
|---|---|---|
| **Primary** | `expo-file-system` JSON files | Notes, folders, AI summaries, topic cache, bookmarks |
| **Secondary** | `AsyncStorage` | Theme, streak, settings, haptics preference |

### Why FileSystem over AsyncStorage?
- AsyncStorage has a 2–6MB per-item limit — large note archives exceed this
- JSON parse hangs on large payloads were observed
- FileSystem avoids IPC overhead for large strings

### Cache Integrity
Every AI artifact is keyed by `djb2Hash(content)`:
```typescript
function hashContent(str: string): string {
    let hash = 5381;
    for (let i = 0; i < str.length; i++) {
        hash = ((hash << 5) + hash) ^ str.charCodeAt(i);
        hash = hash & hash;
    }
    return (hash >>> 0).toString(36);
}
```
Hash mismatch → stale cache → regenerate. Prevents serving outdated summaries.

---

## 10. Performance Metrics

| Operation | Latency |
|---|---|
| Structural parsing | < 10ms |
| Jaccard semantic shift check | < 50ms |
| Extractive summarization (BM25+LexRank) | 300–800ms |
| LLM model load into memory | 3–8 seconds |
| Block-level LLM summary (512 tokens) | 60–120 seconds |
| Whole-note LLM summary (512 tokens) | 60–120 seconds |
| Summary cache read (cached hit) | < 5ms |

---

## 11. Optional: Cloud Intelligence (`lib/aiIntelligence.ts`)

When a Gemini API key is provided in Settings, the app can optionally use cloud inference:

- **Engine**: `gemini-2.0-flash-lite` (default)
- **Strategy**: Zero-native-code HTTP fetch — no Google SDK required
- **System Prompt**: Optimized for academic organization, producing structured JSON with descriptive titles and deep-dive summaries
- **Privacy note**: Using this mode sends note content to Google's servers

This is an **opt-in secondary path** — the default and primary path remains fully on-device.

---

*Documentation — v2.0 ExecuTorch LLM Architecture*
*Compiled by Rahul Varanasi — © 2026 All Rights Reserved*
