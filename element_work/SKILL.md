---
name: element_work
description: >
  Domain knowledge for StepViz teaching elements — what each UI building block
  does, its data contract, and how to add a new one. Use this skill when you need
  to understand, modify, or create any piece of the guided-learning UI (Concept Card,
  Visualizer, Comprehension Modal, Summary Card, Virtual Teacher, etc.).
applyTo: "frontend/src/components/Learning/**,frontend/src/components/VirtualTeacher/**,frontend/src/hooks/useVirtualTeacher.js,frontend/src/components/VisualizerPanel/**"
---

# SKILL: element_work

## When to invoke
- Adding or modifying a teaching element (any phase component, modal, or panel)
- Debugging a narration or phase-transition issue
- Designing the data shape for a new module section
- Reviewing which element owns which piece of the module JSON

## Key reference file
Always read [element_work/elements.md](element_work/elements.md) first for the full element catalogue.

## Quick data map
| Module JSON key       | Consumed by                        |
|-----------------------|------------------------------------|
| `concept`             | ConceptCard, useVirtualTeacher     |
| `demo`                | SceneManager (autoplay), DemoIntro |
| `prediction`          | PredictionPanel                    |
| `explanation`         | ExplanationPanel, SceneManager     |
| `practice`            | PracticePanel                      |
| `feedback`            | FeedbackBox                        |
| `summary`             | SummaryCard, ComprehensionCheckModal |

## Pitfall: ComprehensionCheckModal MCQ source
**Never** derive the comprehension MCQ from `module.prediction` — that question is already shown in PredictionPanel.  
Use `module.summary.takeaways` → `module.explanation.keyTakeaways` → `module.concept.keyPoints` (in that order).

## Pitfall: full-screen modals
Panels with `data-spotlight="true"` have `isolation: isolate` applied via CSS. Any `position: fixed` inside them is viewport-clipped to the panel. Always use `createPortal(content, document.body)` for modals that must cover the whole screen.

## Pitfall: visualizer narration
`SceneManager` has its own local `playhead` state independent of `useWorkspaceStore.currentStepIndex`. After every playhead update, sync:
```js
useWorkspaceStore.setState({ currentStepIndex: playhead });
```
This is required so `useVirtualTeacher` narrates each step.

## Phase order
gate → concept → demo → prediction → execution → explanation → practice → feedback → summary → done
