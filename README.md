# Dalph

<img width="1264" height="848" alt="image" src="https://github.com/user-attachments/assets/643e0dbc-720d-4736-8d4d-8ef70146be26" />

**Dalph = Dave + Ralph + Roo Code** 🤖


Autonomous AI agent loop for Roo Code that runs repeatedly until all PRD items are complete. Fork of [Geoffrey Huntley's Ralph](https://github.com/AmpAI/amp-coder-ralph), adapted for Roo Code's custom modes and tool system.

⚠️ **This is NOT for token savers.** This is for people who want stuff done autonomously (more or less). Expect high token usage - each story gets fresh context.

🖥️ **Tested primarily on Windows** with Roo Code. Should work on Mac/Linux but YMMV.

---

## What's Different from Ralph?

**Ralph (Amp):**
- Bash script loop
- `jq` for JSON parsing
- Single prompt.md
- Re-analyzes codebase each iteration
- Deletes test utilities
- Files loose in `/tasks/`

**Dalph (Roo Code):**
- Roo Code custom modes
- Native Roo Code tools (Read/Edit Files)
- Separate Orchestrator + Implementer modes
- Merged PRD Creator mode + PRD JSON conversion
- Analyzes once, caches in `context.md`
- More "test focused"
- Preserves tests in `test_tools/` folder
- Per-task folders: `/tasks/[task-name]/`

---

## Flow

1. **dalph-prd-creator** - Ask clarifying questions, generate PRD markdown, convert to prd.json

2. **dalph-orchestrator** - Detect stack ONCE and create context.md, pick next story from prd.json, create subtask for implementer, process result and update prd.json, repeat until all stories pass or 3+ blocked

3. **dalph-implementer** (called as subtask per story) - Read context.md (short file, not whole codebase), implement ONE story, verify each acceptance criterion, save test tools to test_tools/, commit and report back

---

## Setup

### 1. Install Custom Modes in Roo Code

Go to **Roo Code → Settings → Modes → Create New Mode** and add three modes:

- **🏭 DALPH: PRD Creator** (slug: `dalph-prd-creator`) - see `dalph-prd-creator/dalph-prd-creator-info.md`
- **🏭 DALPH: Orchestrator** (slug: `dalph-orchestrator`) - see `dalph-orchestrator/dalph-orchestrator-info.md`
- **🏭 DALPH: Implementer** (slug: `dalph-implementer`) - see `dalph-implementer/dalph-implementer-info.md`

Copy the Role Definition, Short Description, When to Use, and Custom Instructions from each file.

### 2. Create 'tasks' Folder in your project directory


## Usage

### Step 1: Create PRD

Switch to `dalph-prd-creator` mode:

```
Create a PRD for [describe your feature]
```

Answer the clarifying questions (format: "1A, 2B, 3C"), review generated PRD, confirm JSON conversion when asked.

Output structure:
```
tasks/[task-name]/
  prd-[task-name].md
  prd.json
  progress.txt
```

### Step 2: Run Orchestrator


- Switch to `dalph-orchestrator` mode.

- Set  `"BRRR" / "YOLO"` mode ON to enable autonomous execution.

- Run the orchestrator with your task:
```
run @/tasks/[task-name]/prd.json - (task: @/tasks/[task-name]/prd-[task-name].md )

```

Orchestrator will detect project stack, create context.md, then loop through stories - creating implementer subtasks, processing results, updating prd.json until done or 3+ stories blocked.

(Step 2.5 - go grab a coffee while it works. Or two. Maybe three.)

### Step 3: Done

When all stories pass: `<promise>COMPLETE</promise>`

If blocked: `<promise>BLOCKED - 3+ stories failed</promise>`

---

## Task Folder Structure

```
tasks/[task-name]/
  prd-[task-name].md   - PRD document (human-readable)
  prd.json             - Stories + progress (machine-readable)
  progress.txt         - Implementation log + learnings
  context.md           - Short context for implementers (~500 tokens)
  test_tools/          - Preserved test utilities
```

---

## Key Concepts

### Small Stories
Each story must be completable in ONE iteration. Split big stories. Good: "Add status column to database". Bad: "Build entire notification system".

### Acceptance Criteria = Tests
Each criterion must be verifiable. Implementer will actually test each one. Good: "Button shows confirmation dialog before deleting". Bad: "Works correctly".

### Context Efficiency
Orchestrator analyzes stack once, saves to context.md. Implementers read only context.md (~500 tokens), not whole codebase. New patterns flow back through orchestrator to context.md.

### Test Tools Preserved
Implementers save verification scripts to test_tools/. These are kept for debugging, not deleted.

---

## Checking Progress

```bash
cat tasks/[task-name]/prd.json      # story status
cat tasks/[task-name]/progress.txt  # implementation log
cat tasks/[task-name]/context.md    # discovered patterns
git log --oneline -10               # git history
```

---

## Tips

- Use it with Roo Code Cloud to grab more coffees while it running.

---

## Credits

- Original - Ralph by [Geoffrey Huntley](https://ghuntley.com/ralph)
- For of [snarktank/ralph](https://github.com/snarktank/ralph)
- Adapted for [Roo Code](https://github.com/RooVetGit/Roo-Code) by theyv

---

## License

MIT
