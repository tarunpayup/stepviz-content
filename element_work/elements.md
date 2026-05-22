# StepViz Teaching Elements — Reference

Each element below is a discrete UI/UX building block in the guided learning flow.

---

## 1. Code Visualizer
**Where:** `VisualizerPanel/SceneManager.jsx`  
**What it does:** Renders a frame-by-frame animation of code execution — variable memory, output, loop state, and per-step explanations.  
**Key behaviour:**
- Local `playhead` state drives animation; synced to `useWorkspaceStore.currentStepIndex` via useEffect so the Virtual Teacher narrates each step.
- Plays automatically on `demo` phase; manual-only on `execution` and `explanation` phases.
- Feeds steps through `virtualMemory.applyStep()` to produce a `MemorySnapshot` per frame.
- `explanationEngine.js` post-processes the step array and attaches a structured `explanation` object to every step before the visualizer receives them.

---

### Step Schema — Full Reference

Every execution step is a plain object with a `type` discriminator and a `line` number. The fields below are the **canonical schema** used by `virtualMemory.js`, `explanationEngine.js`, and `SceneManager.jsx`. When authoring content for any new language, every step MUST conform to this schema.

---

#### `declare` — Variable declaration / first assignment
> A named memory slot is created and given an initial value.

```jsonc
{
  "type": "declare",
  "line": 3,          // 1-based source line number
  "var": "count",     // variable name (identifier)
  "value": "0",       // initial value — always a STRING (the VM stores strings)
  "dataType": "int"   // language-level type label (int, str, float, bool, …)
}
```

**What the visualizer shows:** A new variable card appears in the memory panel with a green "new" badge. The source line is highlighted.  
**Explanation fields auto-generated:**
- `what` — "Declaring variable 'count'"
- `why` — "Allocating a memory slot of type int"
- `change` — "count = 0 (new slot)"
- `intent` — "Reserve a named slot in memory to hold a value we will use later"
- `misconception` — "A declaration does not run logic — it only creates the container"

**Multi-language notes:**
| Language | `dataType` examples | Notes |
|---|---|---|
| Java | `int`, `double`, `boolean`, `String` | Strongly typed; always provide `dataType` |
| Python | `int`, `float`, `str`, `bool`, `list` | Type is inferred; `dataType` used for display only |
| JavaScript | `number`, `string`, `boolean`, `object` | Use `let`/`const`/`var` kind in a `kind` field (optional) |
| SQL | — | No variable declarations; use `assign` for SET / INTO |
| C++ | `int`, `double`, `char`, `bool` | Same as Java; add `kind: "stack"` or `kind: "heap"` if desired |

---

#### `assign` — Variable reassignment
> An existing memory slot receives a new value.

```jsonc
{
  "type": "assign",
  "line": 7,
  "var": "count",     // must already exist in memory (from a prior declare)
  "value": "5"        // new value (string)
}
```

**What the visualizer shows:** The existing variable card flashes amber and the value updates with a slide transition. The old value is shown briefly as `prev`.  
**Explanation fields auto-generated:**
- `what` — "Updating variable 'count'"
- `why` — "Reassigning the existing slot — runtime overwrites the previous value"
- `change` — "0 → 5"
- `intent` — "Store a new computed value so the program can use it in upcoming steps"
- `misconception` — "The old value (0) is gone permanently — the language does not keep history"

---

#### `print` — Console output
> A value is sent to standard output (does not change memory).

```jsonc
{
  "type": "print",
  "line": 9,
  "value": "Hello World"   // resolved string to display in the output panel
}
```

**What the visualizer shows:** A new line appears in the output console panel.  
**Explanation fields auto-generated:**
- `what` — "Printing to console"
- `why` — "Sending the resolved expression to standard output"
- `change` — "output ← \"Hello World\""
- `intent` — "Display the current state of an expression so we can observe progress"
- `misconception` — "print does not change memory — it only reads and displays a value"

**Multi-language notes:**
| Language | How to trigger | `value` |
|---|---|---|
| Java | `System.out.println(x)` | Resolved string value |
| Python | `print(x)` | Resolved string value |
| JavaScript | `console.log(x)` | Resolved string value |
| SQL | `SELECT` result set | One row = one `print` step |

---

#### `loop_start` — Loop initialization
> The loop structure opens; the control variable is about to be set up.

```jsonc
{
  "type": "loop_start",
  "line": 5,
  "kind": "for",          // "for" | "while" | "do_while" | "forEach"
  "condition": "i < 5",   // the loop guard expression as authored (symbolic)
  "var": "i"              // loop control variable name (null for while/forEach)
}
```

**What the visualizer shows:** A loop scope indicator opens. The loop variable (if any) is highlighted.  
**Explanation fields:** Marks loop body as not yet running; condition check is next.

---

#### `loop_check` — Loop condition evaluation
> The guard expression is evaluated; if false, the loop ends.

```jsonc
{
  "type": "loop_check",
  "line": 5,
  "condition": "i < 5",
  "result": true,          // boolean — true = body runs, false = loop exits
  "iteration": 2           // 0-based count of how many times body has run so far
}
```

**What the visualizer shows:** The condition is displayed with current variable values substituted in. A green ✓ (body runs) or red ✗ (loop exits) badge appears.  
**Explanation fields auto-generated:**
- `what` — "Loop condition holds" or "Loop condition fails"
- `why` — "Evaluating i < 5 with i=2 on iteration 2"
- `change` — "iteration 2 body will run" or "loop exits"
- `misconception` — "The loop does not remember past iterations — it re-evaluates fresh every time"

---

#### `loop_end` — Loop close
> The loop scope closes; `activeLoop` is cleared from memory.

```jsonc
{
  "type": "loop_end",
  "line": 8
}
```

**What the visualizer shows:** The loop scope indicator closes. The loop control variable is unmarked.

---

#### `call` — Function / method call *(planned)*
> Records that a named function is being called (enables call stack display in future).

```jsonc
{
  "type": "call",
  "line": 12,
  "name": "add",           // function name
  "args": ["3", "4"],      // resolved argument values (strings)
  "returnVar": "result"    // variable that receives the return value (optional)
}
```

**Status:** Schema defined; not yet consumed by `virtualMemory.js`. Implement when call-stack visualization is needed.

---

#### `return` — Function return *(planned)*

```jsonc
{
  "type": "return",
  "line": 14,
  "value": "7"             // resolved return value
}
```

---

#### `condition_check` — If/else branch *(planned)*
> Records a branch decision for if/else/switch visualization.

```jsonc
{
  "type": "condition_check",
  "line": 6,
  "expression": "x > 0",
  "result": true,
  "branch": "if"           // "if" | "else" | "else_if"
}
```

---

### `explanation` object — attached by `explanationEngine.js`

After parsing, `explanationEngine.js` post-processes the step array and attaches an `explanation` object to every step. The visualizer and Virtual Teacher read from this object — **never** compose explanation text inline in a component.

```jsonc
{
  "what": "Short label — what happened on this line",
  "why": "One sentence — the runtime reason this step exists",
  "change": "Compact diff — memory before → after (e.g. 'count = 0 → 5')",
  "intent": "Pedagogical purpose — what the learner should understand",
  "misconception": "One common wrong mental model this step corrects"
}
```

For **manually authored content** (non-parsed modules — Python, SQL, DSA, etc.), add the `explanation` object directly in the step JSON. The engine only generates it when steps are produced by a parser.

---

### Full authored step example (Python, manual)

```jsonc
{
  "type": "declare",
  "line": 1,
  "var": "name",
  "value": "Alice",
  "dataType": "str",
  "explanation": {
    "what": "Storing the string 'Alice' in a variable called name",
    "why": "Python creates a label that points to the string object in memory",
    "change": "name = 'Alice' (new binding)",
    "intent": "Create a reusable reference to a value so we can use it later",
    "misconception": "In Python, variables are labels (references), not boxes — name points TO the string, it doesn't contain it"
  }
}
```

---

### Parser → Step pipeline (per language)

```
Source code
    │
    ▼
Language Parser  (e.g. javaParser.js / pythonParser.js / sqlParser.js)
    │  produces raw steps with type, line, var, value, dataType
    ▼
explanationEngine.decorateSteps(steps)
    │  adds .explanation to every step via before/after VM snapshot diff
    ▼
standardizeSteps(steps)         ← SceneManager normalizes step types
    │
    ▼
VirtualMemory.applyStep(snapshot, step)   ← produces MemorySnapshot per frame
    │
    ▼
SceneManager renders frame
```

To add a **new language parser**, create `frontend/src/engine/{lang}Parser.js` exporting:
```js
export function parse{Lang}Code(code: string): Step[]
```
The output MUST use only the step types documented above. The explanation engine and visualizer will work without any other changes.

---

## 2. Concept Card
**Where:** `LessonPanel/ConceptCard.jsx`  
**What it does:** The opening card — introduces the topic before any code is seen.  
**Data shape:**
```jsonc
{ headline, explanation, keyPoints: ["string", ...] }
```
**Auto-voice:** `useVirtualTeacher` narrates `concept.explanation` on phase enter.

---

## 3. Demo Intro
**Where:** Phase `demo` — `LessonPanel` renders code + runs the visualizer in autoplay.  
**What it does:** Shows code executing on its own so learners see the pattern before they need to predict or interact.  
**Data shape:**
```jsonc
{ code, focusLine, expectedHighlights: [int] }
```

---

## 4. Prediction Panel
**Where:** `LessonPanel/PredictionPanel.jsx`  
**What it does:** MCQ asked *before* learner sees execution. Primes anticipation.  
**Data shape:**
```jsonc
{ question, options: ["string", ...], correctIndex: int, explanation }
```
**Note:** Never reuse this MCQ in `ComprehensionCheckModal` — it is already shown here.

---

## 5. Step Explanation Panel
**Where:** `LessonPanel/ExplanationPanel.jsx`  
**What it does:** Per-step "what / why / change" sidebar that appears alongside the visualizer.  
**Data source:** `explanationEngine.js` derives explanations from the standardized step array.  
**Data shape:**
```jsonc
{ what, why, change, codeContext }
```

---

## 6. Practice Panel
**Where:** `LessonPanel/PracticePanel.jsx`  
**What it does:** A code editor challenge — learner writes / completes code that satisfies validation rules.  
**Data shape:**
```jsonc
{ starterCode, solutionCode, validationRules: [{ type, target, message }] }
```
**Phases:** `practice` and later re-entry via "Practice Session" in SummaryCard.

---

## 7. Feedback Box
**Where:** `LessonPanel/FeedbackBox.jsx` (rendered by `LearningController` after practice submission)  
**What it does:** Shows pass/fail with hints and detected errors.  
**Data shape:**
```jsonc
{ passed: bool, retryHints: ["string"], detectedErrors: ["string"] }
```

---

## 8. Summary Card
**Where:** `components/Learning/SummaryCard.jsx`  
**What it does:** Final phase — celebrates completion, shows takeaways, offers Next / Recap / Practice.  
**Props:** `{ summary, module, onNext, onRestart, onPractice }`  
**Data shape for `summary`:**
```jsonc
{ title, takeaways: ["string"], nextTopicId }
```
**Inline "Practice Session":** expands an MCQ block derived from module content (max 3 Qs). Allows the learner to self-test without leaving the summary screen.

---

## 9. Comprehension Check Modal
**Where:** `components/Learning/ComprehensionCheckModal.jsx`  
**What it does:** Gates the Summary Card — asks "Did you understand?" before unlocking it.  
**Portal:** Renders via `createPortal(content, document.body)` so it is truly full-screen even inside CSS-isolated panels.  
**MCQ source:** `summary.takeaways` (first entry is correct; distractors from remaining takeaways + concept keyPoints). Falls back to `explanation.keyTakeaways` then `concept.keyPoints`.  
**Never** uses `prediction.options` — that question is already shown in PredictionPanel.

---

## 10. Learning Mind Map
**Where:** `components/Learning/LearningMindMap.jsx`  
**What it does:** A horizontal 8-phase visual progress strip pinned to the top of the lesson panel.  
**Phases:** gate → concept → demo → prediction → execution → explanation → practice → summary  
**Visuals:**
- Green ✓ circle = completed phase
- Pulsing yellow ring = current phase
- Dim gray = upcoming
- Animated fill-bar connectors between nodes
- Topic subtitle under each node sourced from `module`

---

## 11. Virtual Teacher
**Where:** `hooks/useVirtualTeacher.js` + `engine/teacherVoice.js`  
**What it does:** Text-to-speech narration triggered by phase changes and step changes.  
**Narration conditions:**
- Phase change (`concept`, `demo`, `prediction`, `execution`, `explanation`, `practice`, `summary`) — one sentence intro
- Step change in phases `execution | demo | explanation` — reads `step.explanation` or auto-generated fallback
**Toggle:** `FloatingTeacherButton` in workspace; `teacherModeEnabled` in `useWorkspaceStore`.

---

## 12. Floating Teacher Button
**Where:** `components/VirtualTeacher/FloatingTeacherButton.jsx`  
**What it does:** Persistent bottom-center pill button in the workspace.
- In **learning mode**: clicking mutes/unmutes voice (teacher always stays on)
- In **playground mode**: toggles teacher on/off entirely
- When active: yellow glow + animated pulse dot
- On hover: expands to show "Explanation" / "Mute Voice" / "Turn Off" label

---

## 13. Virtual Teacher Badge
**Where:** `components/VirtualTeacher/VirtualTeacherBadge.jsx`  
**What it does:** Small animated avatar / badge that appears when the teacher is narrating.

---

## 14. Exploration Mode (Sandbox)
**Where:** Accessed after guided flow is complete or via teacher toggle in playground.  
**What it does:** Free editor — no phase gating, no teacher forcing. Learner can run any code.

---

## Adding a New Element — Checklist

1. **Define the data shape** in `docs/context.md` and the module's JSON schema.
2. **Create the React component** in the appropriate `components/` subdirectory.
3. **Add a phase entry** in `LearningController.jsx` if it's a new learning phase.
4. **Wire narration** in `useVirtualTeacher.js` (`buildNarration` switch statement + step-narration condition if applicable).
5. **Update LearningMindMap** node list if a new phase is introduced.
6. **Update this file** with the element's definition, data shape, and key behaviour.
