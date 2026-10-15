# GAIA Archtree — Design

Status: topology + operating laws are canon and in daily use; enforcement lives in the ignition skill (prompt level), not in daemon code. See *Implemented vs designed* in [README.md](README.md).

## 1. Problem

- **Context rot** — one agent working a large task accumulates stale context; quality decays with length.
- **Self-agreement** — a model grading its own output tends to confirm it; "done" claims go unverified.
- **Single persona, single perspective** — one model/persona yields one distribution of answers.
- **Fragile fan-out** — background workers that die silently, or whose results are lost, make large swarms untrustworthy.

## 2. Core structure

### 2.1 Twin adversarial binary trees (敵対二進木 ×2)

- **陽 tree — BUILD.** Produces the artifact.
- **陰 tree — ADVERSARY.** Mirror of the build tree; its output is a verdict, never a fix. It exists to refine the 陽 output by destroying whatever cannot survive attack.
- Both trees are **binary**: apex → 2 roots → 2 children each → … to **depth 4**, i.e. up to **16 nodes per side**, 32 + 32 at full build. Binary keeps spans of control small (each node briefs and judges only two children) and makes the shape predictable.
- Mirroring is a *presentation* as much as a topology: every node of the build tree has an adversarial counterpart; the same graph is seen from both sides. (In the visualizer the tree is literally drawn twice — 陽 crown up, 陰 root-half mirrored below; a node belongs to both halves, so polarity is a rendering property, not a data field.)

### 2.2 Apex: one persona, deepest memory

- Exactly one persona sits at the apex. It owns long-term memory and the relationship with the human, decomposes the goal, routes work to roots, and **verifies results itself** before anything is called done.
- No other node needs that memory. Depth below the apex is disposable.
- Duties of the apex (from the skill): run a *recon wave* before full build if no feature board exists (the checklist becomes the season board); reroute a dead lane (provider error, timeout) rather than restart the tree; verify claims with its own grep/test; adopt the running children of a dead root instead of rebuilding.

### 2.3 Clean-context children speaking gaiago

- Every node below the apex is a **fresh instance with an empty context**, given a self-contained spec. Nothing is inherited implicitly — so every spec carries the standing laws ("laws don't teleport").
- Instances communicate and reason in **gaiago**: a terse notation of fragments, arrows and pointers — no filler prose. It keeps inter-node traffic small, makes briefs state-once-point-after, and keeps the child's reasoning legible to its parent.
- Models are **mixable per node** (different vendors, sizes, personalities). The tree is model-agnostic; diversity is a feature (see 2.4).
- Roots spawn their own children (`summon`), so the tree builds itself recursively; the apex only launches the roots.

### 2.4 Circle of seasons (季節の輪)

- Roles **rotate between waves**: a model/persona that built in one wave attacks or builds a different slice in the next. Effects:
  - **Clean context** — lanes are short-lived; nobody drags a wave's baggage into the next.
  - **Persona diversity** — the same board is seen by different models, so correlated blind spots get covered.
  - **Bounded adversaries** — an adversary pass is time-boxed per wave (tens of minutes, ≤12 children) rather than an endless watch loop.
- Rotation is operated by the apex; there is no scheduler (yet).

### 2.5 Branch-at-root

While a tree runs, an orthogonal task becomes a **new root branch** directly under the apex (one agent), so it never steals context from an existing branch. In the daemon this is `/archtree add-root`.

## 3. Operating laws (distilled from the ignition skill)

Every node spec repeats these — they were learned from failures, not invented up front:

1. **Acceptance that can't be hollowed** — name a currently-failing test that must pass through the *production* path. Structural/"validity" tests that pass on empty shells don't count.
2. **No unverified "done"** — a feature without a real boot and a real test is `UNVERIFIED`. The apex re-verifies every root's claim before it reaches the board.
3. **Adversary mandate** — watch the commit log; per feature: boot, exercise, hit edge cases (collision, sustained load, out-of-bounds, physics stress), check architecture (layer separation, constants grep); file tickets with repro into a committed ledger. **Verdicts only, no fixes.**
4. **Adversary is bounded and restart-proof** — first action is a branch plus an empty ledger section, committed; never an open-ended watch loop.
5. **Isolation** — every child works in its own git worktree/branch; children squatting the caller's checkout switched its branch in practice. Landing goes through a dedicated merge worktree with fast-forward only.
6. **No pragma escapes** — type-suppression comments / `as any` are forbidden in atoms; reroute the cut, not the compiler. Acceptance: the count never grows.
7. **Commit every stage** — a lane that times out loses its room, not its committed work; salvage from disk before relaunching.
8. **Orphan adoption** — if a root dies, the relaunch reads its transcript and adopts running children; the tree is never restarted wholesale.
9. **Routing by evidence** — roots go to lanes known to hold up under the spec's density; lanes that die (provider block, timeout) are rerouted and the incident logged.
10. **Spec hygiene** — never embed refusal/policy wording from a failed lane into the next spec (it can trip classifiers at request zero); keep specs to fragments + paths.

## 4. Lifecycle of a tree

```
ignite ─► recon wave (board of features = season board)
      ─► launch 陽 root(s) + 陰 root(s)           # apex: summon
      ─► roots recursively summon children       # depth ≤ 4
      ─► children commit per stage, report upward # result notes in parent rooms
      ─► 陰 files tickets with repro                # ledger committed
      ─► 陽 fixes, 陰 re-verifies                    # until zero open findings
      ─► apex verifies independently, then says done
      ─► next wave: rotate roles/models (季節の輪)
```

## 5. Mapping onto gaia-daemon

No dedicated tree object exists in the daemon. A tree **is** a set of summon rooms linked by `parentRoomId`.

- **Launch** — `SummonCoordinator.launch` creates a child room (incognito: excluded from recall and episodic memory), stamps `parentRoomId` and the pending delivery *before* the child's first turn, optionally creates an own git worktree, derives the child's trust tier from the caller (untrusted callers' children run sandboxed — data flow, not an approval gate), and returns the room id immediately. A per-room cap bounds concurrent running children.
- **Fan-out** — the `summon` tool accepts a `whales` list; each entry becomes its own parallel sub-room. The call returns at once; the caller is re-invoked as results land.
- **Upward flow** — on completion the result is posted into the parent as a collapsed, worker-authored note; `deliver: "turn"` also steers the running parent turn (or starts one). Failures are delivered the same way. Pending delivery is persisted on the child's state, and `recoverUndelivered` re-arms it after a restart.
- **Steering** — `gaia resume <roomId> "<msg>"` injects follow-up orders into a sub-room; resume records are tracked to delivery like first turns. Lanes are steerable, never fire-and-forget.
- **Observability** — `gaia summon --status` classifies lanes as running / idle / completed-undelivered / dead (with last event, delivered?, dirty worktree); the `/archtree` view renders the same parent links in 3D.
- **Roots** — `addArchtreeRoot` (`src/services/archtree.ts`): validate task+agent → load installed `gaia-archtree` skill (refuse to launch if absent/empty) → prompt = skill + root-branch header ("scope = assigned task only; preserve existing tree; skill laws apply to every delegated child") + task → ordinary `summon` with `ownWorktree` defaulting to true and `deliver: turn|note` depending on whether an agent or a human asked. Entry points: slash command, CLI, HTTP harness route, UI route.

Small excerpt of the entire root path:

```ts
const skill = await loadSkillText(request.workspace, ["gaia-archtree"]);
if (skill.diagnostics.length || !skill.text.trim()) throw new Error("Archtree skill unavailable");
const prompt = [skill.text, "# Root branch\nScope → assigned task only · existing tree → preserve · skill laws → every delegated child", `# Task\n${task}`].join("\n\n");
return coordinator.summon(request.parentRoomId, agentId, prompt, { ownWorktree: request.ownWorktree ?? true, /* deliver: turn | note */ });
```

## 6. Visualizer (`/archtree`)

A dependency-free WebGL2 view over the room tree (`web/src/archtree/*`: `model` → `layout` → `renderer` behind a `RendererPort`). Tree edges come from the existing snapshot (`parentRoomId`), so it adds no server route or data model.

- Status derived from existing fields: running → active; recently finished → done; stale → withered; else idle. A true `failed` bit does not exist yet.
- 陽 crown drawn up, 陰 roots mirrored down with an inverted material language (dead lanes pale above, vivid below).
- Orbit/zoom/hover; click a node to open that room, where it is steerable like any room.
- All tunables are named parameters with defaults (no magic numbers); an alternative renderer is an interface, and an unavailable one reports loudly rather than silently falling back.

## 7. Evidence so far

- Root lifecycle: a compiled-binary probe launched three live roots under one coordinator (two via the command with an explicit agent, one with the default agent), confirmed same parent link, running state, durable delivery record, installed skill present in each root's first task, and delivery of all three completions. Test gates: archtree and summon suites green.
- Operational use: refactor-scale trees (about a dozen build roots, five adversary passes, ~170 commits, two dozen adversary findings opened and closed) and several smaller build/adversary pairs; the adversary caught, among others, an interlock that wrote when it should have refused.
- Not proven: recursive spawning, routing choices and descendant compliance in a controlled experiment; the lifecycle probe covered roots only.

## 8. Open design questions

- Enforce shape (binary, depth ≤ 4, 陽/陰 pairing) in the daemon rather than in the skill?
- Scheduler for 季節の輪 rotation, with rotation policy as data.
- First-class `failed` outcome on summon delivery, surfaced in the tree view.
- Measuring adversary yield (tickets per build commit, escaped defects) to tune tree width.
- Cross-tree memory: what, if anything, flows from a finished wave back to the apex beyond the verified result.
