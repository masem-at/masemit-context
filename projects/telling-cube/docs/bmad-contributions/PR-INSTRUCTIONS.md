# BMad Method PR Instructions

**Author**: Mario Semper (masem1899)
**Organization**: masem-at
**Target Repo**: https://github.com/bmad-code-org/BMAD-METHOD.git

---

## PRs Ready to Submit

### PR #1: Ring of Fire (ROF) Sessions
**Branch**: `feature/ring-of-fire-sessions`
**File**: `docs/ring-of-fire-sessions.md`
**Description**: Multi-agent parallel collaboration feature

### PR #2: Agent Task Pre-Flight Protocol
**Branch**: `feature/agent-preflight-protocol`
**File**: `docs/agent-preflight-protocol.md`
**Description**: Safety framework for high-risk agent tasks

---

## How to Submit (Manual Method)

### Step 1: Fork the Repository

```bash
# Go to https://github.com/bmad-code-org/BMAD-METHOD
# Click "Fork" button
# Choose masem-at organization (or masem1899 personal)
```

### Step 2: Clone Your Fork

```bash
cd ~/projects
git clone https://github.com/masem-at/BMAD-METHOD.git  # or masem1899/BMAD-METHOD
cd BMAD-METHOD
git remote add upstream https://github.com/bmad-code-org/BMAD-METHOD.git
```

### Step 3: Create PR #1 (Ring of Fire)

```bash
git checkout -b feature/ring-of-fire-sessions

# Copy the file
cp /tmp/bmad-temp/docs/ring-of-fire-sessions.md docs/

git add docs/ring-of-fire-sessions.md
git commit -m "feat: Ring of Fire (ROF) Sessions - Multi-agent parallel collaboration"

git push origin feature/ring-of-fire-sessions

# Then go to GitHub and create PR
```

### Step 4: Create PR #2 (Pre-Flight Protocol)

```bash
git checkout main
git checkout -b feature/agent-preflight-protocol

# Copy the file
cp /tmp/bmad-temp/docs/agent-preflight-protocol.md docs/

git add docs/agent-preflight-protocol.md
git commit -m "feat: Agent Task Pre-Flight Protocol - Safety framework"

git push origin feature/agent-preflight-protocol

# Then go to GitHub and create PR
```

### Step 5: Open PRs on GitHub

1. Go to your fork: `https://github.com/masem-at/BMAD-METHOD` (or masem1899)
2. Click "Pull Requests" → "New Pull Request"
3. Select your branch
4. Fill in PR template (use commit message + description from markdown files)
5. Submit!

---

## Alternative: Use Prepared Commits

The commits are already prepared in `/tmp/bmad-temp/`:

```bash
cd /tmp/bmad-temp
git log --oneline
# 10dc25f4 feat: Ring of Fire (ROF) Sessions...
# [previous commit] feat: Agent Task Pre-Flight Protocol...

# You can push these directly to your fork
git remote add myfork https://github.com/masem-at/BMAD-METHOD.git
git push myfork feature/ring-of-fire-sessions
git push myfork feature/agent-preflight-protocol
```

---

## PR Descriptions (Copy-Paste Ready)

### PR #1 Title
```
feat: Ring of Fire (ROF) Sessions - Multi-agent parallel collaboration
```

### PR #1 Description
```markdown
## Summary
Introduces **Ring of Fire (ROF) Sessions** - a feature enabling multi-agent collaborative sessions that run in parallel to the user's main workflow.

## Problem
Current BMad requires sequential agent interaction. Users can't delegate complex multi-perspective analysis while continuing other work.

## Solution
New `*rof` command allows users to:
- Start scoped agent collaboration sessions
- Continue working in parallel
- Get consolidated reports (brief/detailed/live)
- Maintain control via approval gates

## Real-World Validation
- **Origin**: tellingCube project (masemIT e.U.)
- **Tested**: Successfully ran multi-agent planning sessions
- **User Feedback**: "This is exactly what I needed - multiple perspectives without orchestrating every conversation"

## Documentation
See `docs/ring-of-fire-sessions.md` for full specification, use cases, and implementation details.

## Related
- Addresses parallel workflow bottleneck
- Enables complex multi-agent analysis
- Maintains user control and safety
```

---

### PR #2 Title
```
feat: Agent Task Pre-Flight Protocol - Safety framework for high-risk tasks
```

### PR #2 Description
```markdown
## Summary
Introduces **Agent Task Pre-Flight Protocol** - mandatory safety checks for high-risk agent tasks to prevent factual errors, trademark violations, and privacy breaches.

## Problem
Agents can create public content (marketing, legal docs) without verifying:
- Actual product features/pricing (leads to wrong claims)
- Trademark compliance rules (legal risk)
- Privacy guidelines (personal info leaks)

**Real failure case**: Marketing agent created LinkedIn posts with wrong pricing, trademark violations, and privacy leaks - all preventable with verification.

## Solution
Pre-flight protocol requiring agents to:
1. Read critical guidelines before creation
2. Verify facts against actual codebase (not assumptions)
3. Cross-agent fact-checking
4. User approval gates for high-risk outputs

## Real-World Validation
- **Origin**: Painful lessons from tellingCube project
- **Before Protocol**: 5 critical errors in marketing content
- **After Protocol**: Zero errors, factually accurate
- **User Feedback**: "I don't want to repeat the ChatGPT hallucination nightmare. This protocol gives confidence."

## Documentation
See `docs/agent-preflight-protocol.md` for:
- Detailed protocol specification
- CRITICAL-GUIDELINES.md template
- Implementation examples
- High-risk task categories

## Related
- Prevents agent "hallucination" of facts
- Protects legal compliance
- Builds user trust
```

---

## Files Location

All files are in:
- `/tmp/bmad-temp/` (git repo with commits ready)
- `C:\Users\mario\Sources\dev\telling-cube\docs\bmad-contributions\` (source files)

---

**Next**: Fork repo, push branches, create PRs! 🔥
