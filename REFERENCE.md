# Deep Dive — Reference

## File structure per topic

```
Deep Dives/<topic>/
├── context.md       ← purpose, depth, angles (written on first session)
├── topic-map.md     ← dependency tree with status markers
└── session-1.md     ← structured session notes (one file per session)
```

---

## topic-map.md format

```
<Topic>
├── Foundations
│   ├── Concept A (prerequisite for B, C)          [x]
│   └── Concept B                                  [~]
├── Core mechanics
│   ├── Concept C                                  [ ]
│   └── Concept D                                  [ ]
├── Advanced / applied
│   ├── Concept E                                  [ ]
│   └── Concept F                                  [ ]
└── Adjacent knowledge (useful but not required)
    └── Concept G                                  [ ]
```

**Node status markers**:
| Marker | Meaning |
|--------|---------|
| `[ ]` | Not started |
| `[~]` | Partially covered |
| `[x]` | Covered and retained (passed quiz) |
| `[?]` | Covered but weak (failed or uncertain) |

---

## session-N.md format

```markdown
# Session <N> — <YYYY-MM-DD>

## What we covered
- Subtopic A
- Subtopic B

## Concept summaries

**Concept A**: <concise plain-language explanation>

**Concept B**: <concise plain-language explanation>

## Cue cards

**Q**: <question>
**A**: <answer>

**Q**: <question>
**A**: <answer>

## Open questions
- <gap or thread to pursue next session>
- <another open question>

## Confidence map

| Subtopic | Claude score | User override | Status |
|----------|-------------|---------------|--------|
| Concept A | 4/5 | — | [x] |
| Concept B | 2/5 | 3/5 | [?] |
```

---

## context.md format

```markdown
# <Topic> — Context

**Purpose**: <why the user is learning this>
**Target depth**: <conversational / working knowledge / expert mastery>
**Priority angles**: <specific sub-areas to focus on>
**Started**: <YYYY-MM-DD>
```
