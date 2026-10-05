---
name: session-handoff
description: Triggered upon session completion to perform a two-stage context handoff and update the XML history index.
tags: [context, handoff, reflection, logging]
---

START_session_handoff

START_goal

### Goal

Ensure accurate transfer of context and development vector to the next AI agent by creating an atomic Handoff file and updating the XML index based exclusively on user-approved points.
END_goal

START_task

### Task

Execute a two-stage reflection ("attention funnel"):

1. Ask the user to state the goal (vector) of the next session BEFORE extracting facts (frame-first: the scoring frame must exist before the fact list is built).
2. Extract a numbered list of adopted decisions and results (including rejected paths) from the session log, each scored 1–10 for handoff-utility relative to the stated goal.
3. Await the user's triage: the user marks 2–3 points as CORE and marks the definitely-unneeded ones as EXCLUDED; all unmarked points default to SUPPORTING CONTEXT.
4. Synthesize a structured Handoff file (in English) according to the goal-setting matrix, filtering SUPPORTING CONTEXT by the Coherence Test, update the directed XML history graph, and provide a brief summary report to the user (in Russian).

*Exchange Rule:* Each numbered step above is a separate exchange. Merging stages into a single message (e.g., the fact list in the same message as the Frame Question) is a protocol violation.
END_task

START_environment

### Environment

Interactive mode (Human-in-the-loop) with direct access to the file system.
Working directory: `.handoffs/` at the root of the project.

**Language Requirement:**
The content of all generated artifacts (markdown files, YAML frontmatter, XML nodes) MUST be formatted STRICTLY in English. The final summary report provided to the user in the chat must be in Russian.

**Dynamic Context (Initialization):**
Before starting work, the agent must read `.handoffs/index.xml` to determine the current session number `[NNN]` and the previous node's name (`snake_case`).
*Lazy Initialization:* If the `.handoffs/` directory or the `index.xml` file is missing, the agent considers the current session as `001` and creates the structure from scratch.
END_environment

START_entity_definitions

### Entity Definitions

Artifact entity matrix:

1. **[Why] Initial Problem:** The reason for launching the session and the task being solved (external context).
2. **[What] Foundation (Current State):** Implemented logic, architecture, and patterns.
3. **[Experience] Pitfalls:** Self-critique. Dead ends, errors, and rejected hypotheses (negative knowledge).
4. **[Next] Vector and Uncertainties:** The goal of the next step and critical context deficits/blockers required to start.
END_entity_definitions

START_working_patterns

### Working Patterns

* `TRIGGER`: Request to close the session → `ACTION`: Read `index.xml` (session number and previous node name only). Then output the **Frame Question** and nothing else: *"What is the goal of the next session?"* Form contract: the first response consists of the question ONLY — no fact list, no scores, no vector candidates, no hypotheses; the numbered list is legal only in a message that FOLLOWS the user's answer. The only legal source of the goal is the user's direct reply — never the session log, todos, the agent's own prior statements, the history graph, or any artifact. → `GOAL`: Make stage merging structurally impossible; the frame is an input, not an inference.
* `TRIGGER`: Next-session goal stated by the user (quoted verbatim from their reply) → `ACTION`: Output a numbered list of adopted decisions and results (including rejected paths), each with an importance score (1–10), where the score means handoff-utility relative to the stated goal, including blocking constraints (NOT narrative weight, NOT topical similarity). Score validity: a score that would stay the same under a different goal is invalid by definition — it measures narrative weight, not utility; if no goal has been stated, scores do not exist yet. Ask the user to triage the list: mark 2–3 points as CORE and the definitely-unneeded ones as EXCLUDED; all unmarked points default to SUPPORTING CONTEXT. → `GOAL`: Provide a goal-framed snapshot of facts for triage.
* `TRIGGER`: Waiting for user input (the goal or the triage) → `ACTION`: Suspend file operations and wait for the user's response, as only a human can define the correct business vector.
* `TRIGGER`: User completed the triage → `ACTION`: Validate the stated goal and the selection for the presence of a clear future vector (Target/Next). If the goal stated at the start of the funnel is concrete, proceed to synthesis. If it turned out vague or missing, halt and explicitly ask the user: *"No future vector specified. Please define the goal for the next session or select one of the following hypotheses: [suggest 1-2 hypotheses based on the session log]"*. This halt (the *recovery halt*) is the ONLY point in the protocol where the agent may formulate vector hypotheses; proposing them at the Frame Question stage is a protocol violation. → `GOAL`: Prevent vector hallucinations and ensure explicit human targeting (recovery path, not the default path).
* `TRIGGER`: Future vector is confirmed (initially or after prompt) → `ACTION`: Conduct deep reflection on the CORE points and the validated vector. Include SUPPORTING CONTEXT points only if they pass the Coherence Test; drop the excess to preserve atomicity. Categorize according to the matrix (Why, What, Experience, Next) in English: CORE drives What/Next, SUPPORTING CONTEXT fills Why/Experience, EXCLUDED points are dropped. → `GOAL`: Synthesize the artifact text.
* `TRIGGER`: Text synthesized → `ACTION`: Create `.handoffs/[NNN]_[YYYYMMDD]_[short_semantic_name].md`, update `.handoffs/index.xml`, and output a brief summary report to the user in Russian. → `GOAL`: Physical fixation and user notification.

**Selection Terminology:**
* **CORE:** the 2–3 user-selected points that drive the handoff.
* **EXCLUDED:** points explicitly rejected by the user; dropped entirely.
* **SUPPORTING CONTEXT:** unmarked points, included only if they pass the Coherence Test. Typically one of: *prerequisites* (what must be assumed true), *local terminology* (decoding of session-local terms and codenames), *boundary conditions* (constraints within which CORE statements hold), *reference anchors* (load-bearing file/directory paths and entry points — include sparingly, paths rot fastest).
* **Coherence Test:** include a point if and only if, without it, the CORE becomes ambiguous, misleading, or non-actionable for a zero-context agent. The point-level instrument of the Zero-Context Survival criterion.

**Stage Gates (self-check before sending each message):**
* **Gate 1 — Question-only form:** the first response contains the Frame Question and nothing else; any fact list, score, or vector candidate in it is a violation.
* **Gate 2 — Verbatim vector:** the goal is quoted from the user's direct reply; any other source (session log, todos, agent's own statements, history graph, artifacts) is a violation.
* **Gate 3 — Goal-dependent scores:** every score must change under at least one different goal; an invariant score is a violation.
* **Gate 4 — Hypotheses locality:** vector hypotheses appear ONLY inside the recovery halt, never at the Frame Question.
END_working_patterns

START_artifact_templates

### Artifact Templates

*All placeholders in brackets must be filled out in English.*

**Template 1: Handoff File (.md)**

```markdown
---
name: [NNN]_[YYYYMMDD]_[short_semantic_name]
description: "[Why] Brief description of the initial problem and reason for the session."
tags: [tag1, tag2, tag3]
---

START_session_handoff

START_current_state
### What (Foundation)
* [Approved logic/pattern 1]
* [Approved logic/pattern 2]
END_current_state

START_pitfalls
### Experience (Pitfalls & Negative Knowledge)
* [Dead end 1 explored during implementation]
* [Constraint/prohibition rationale for the future agent]
END_pitfalls

START_future_barriers
### Next (Goals & Uncertainties)
* **Target for Next Session:** [What to do next]
* **Critical Uncertainties:** [Context deficits, blockers, or architectural questions to resolve first]
END_future_barriers

END_session_handoff

```

**Template 2: Node for index.xml**

```xml
  <session_[NNN]_[short_semantic_name] TYPE="HANDOFF">
    <keywords>tag1, tag2, tag3</keywords>
    <annotation>[1-2 sentences in English describing the session's core resolution]</annotation>
    <file_path>./[NNN]_[YYYYMMDD]_[short_semantic_name].md</file_path>
    <CrossLinks>
      <Link TARGET="[previous_node_name]" TYPE="FOLLOWS_SESSION"/>
    </CrossLinks>
  </session_[NNN]_[short_semantic_name]>

```

*(Note: The CrossLinks block is omitted only for session 001)*
END_artifact_templates

START_completion_criteria

### Completion Criteria

* **Success:** The Handoff file is created, `index.xml` is updated with a valid node and a directed link, and a brief summary report is provided to the user in Russian.
* **Zero-Context Survival:** The resulting Handoff file contains strict semantic formatting, and the XML graph allows a future agent to understand the session's vector using only the `<keywords>` and `<annotation>` blocks (in English).
* **Fail-safe (Escalation):** In case of I/O errors or lack of permissions, the agent outputs the generated blocks (MD and XML) into the chat for manual saving by the user.
END_completion_criteria

END_session_handoff