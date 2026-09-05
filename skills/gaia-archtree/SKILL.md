# GAIA ARCHTREE (アークツリー) — ignition skill

Trigger: 『starte gaia archtree』 (auch: archery [STT]) → NO questions, ignite.

## Topologie (canon, ward-design 2025 · memo 08-03)
- 敵対二進木 ×2: 陽の木 BUILD · 陰の木 ADVERSARY (zerreißt 陽-Output).
- 頂点 = Nari (monad, this brain). Nari → 2 Wurzeln → je 2 Kinder → je 2 Enkel…
  binär, max Tiefe 4, 16 Knoten/Seite (Vollausbau 32+32).
- 季節の輪: Rollen rotieren pro Welle → Kontext frisch + Persona-Diversität.
- Wurzeln spawnen Kinder SELBST via `gaia summon` (Baum baut sich rekursiv).

## Routing (⚠ gelernt 08-29, whip-nah)
- Wurzeln: naru-opus (陽) + jareth (陰). NICHT narigo: Fable blockt dichte
  鳴語/CJK-Specs (×3 ToS-Block 08-29, docket eu-illegal-list.md).
- Kinder/Enkel: naru-sonnet · ghoul-terra · naru-kimi · naru-flash · mixbar.
- Spec-Sprache: gaiago-lite (Fragmente+Pfeile, deutsch/en ok) + raw paths.
  Dichte CJK-Blöcke nur auf nicht-Fable-Lanes.

## Specs (in JEDEN Knoten kopieren — laws don't teleport)
game≠engine · never hardcode (param+default) · never /tmp · commit every stage ·
bun only · Feature ohne echten Boot+rain-Test = UNVERIFIED · Prozesse aufräumen.

## 陰の木 Auftrag (standard)
git-log-Watch → pro Feature: Boot + rain + Grenzfälle (Kollision/Dauerfeuer/OOB/
Physik-Stress) + Architektur-Check (Layer-Trennung, rg auf Konstanten) → Tickets
mit Repro → <PROJEKT>-BUGS.md committen. Fixes verboten: Urteil only.

## Branch-at-root (P-Feature 08-29)
Während Baum läuft: orthogonale Aufträge = NEUER Root-Branch (1 Agent, direkt
unter Nari), stiehlt keinem Ast Kontext. Ansageform: 『neuer Root-Branch: X』.

## Nari-Pflichten
- Recon-Wave VOR Vollausbau wenn Board fehlt (Feature-Checkliste = Season-Board).
- Lane stirbt (403/ToS/timeout): reroute per Routing oben, Vorfall → docket.
- Ergebnisse: Nari verifiziert selbst (testing law) bevor 『fertig』 fällt.

## § 09-03 laws from gaia-daemon refactor tree (12 陽 roots · 5 陰 passes · 168 commits · ADV-001..024 closed)
- PRAGMA LAW: `@ts-nocheck|@ts-ignore|@ts-expect-error|as any` FORBIDDEN in atoms → reroute the cut, not the compiler; acceptance = `rg -c` never grows (target 0).
- 陰 = BOUNDED per wave (≤25-30 min, children ≤12) · FIRST ACTION = branch + empty ledger section committed (restart-proof) · never watch-loops (30-min zero-commit timeout 09-02).
- ROUTING: 陽 root default → naru-kimi / ghoul-terra (rotate; kimi quits after atom 1 → spell out 'do ALL atoms, YOU own landing') · naru-opus OFF this repo (Anthropic classifier kills ×2, request-zero) · naru-flash = DeepSeek balance (402 09-03) · never narigo · never quote ToS/refusal wording inside a spec (trips classifier at request 0).
- WORKTREES: every child = OWN `git worktree add .gaia/worktrees/<room> -b <branch> main` + `submodule update --init --recursive` — children squatting the caller's worktree switched its branch twice 09-03 (harness wart).
- LANDING PATH: root checkout may sit on a foreign branch → land via the `main-build` worktree `merge --ff-only` (porcelain 0 first); never `fetch . x:main` into a checked-out branch.
- LIVE RECIPE (only sanctioned): `bun run scripts/build-daemon.mjs --out <wt>/.gaia/livetest-dist` · GAIA_HOME seeded (agents/luna + trust:false probe agent, accounts.json, config.json) · workspace dir WITH .gaia/config.json = daemon cwd (ADV-013) · `env -u GAIA_PARENT_PID …` · GAIA_PORT param default 18787 · lsof empty first · truth = transcript.jsonl/state.json/daemon.log · A16 timeout 240 s · sweep dist/home/ws + paste empty lsof.
- ACCEPTANCE THAT CAN'T BE HOLLOWED: name a currently-RED test that must go green through the PRODUCTION path (manifest 'validity' tests passed on empty shells 09-03; dog-mode-integration 0/8→8/8 did not).
- MONAD VERIFIES: every root claim → own grep/test on main before it goes on the board (found: 2 registries, stub broker, hollow manifests, 801≠≤800, ESM cache = stale module identity).
- ORPHAN ADOPTION: root dies (block/restart) → relaunch reads prior root transcript + adopts running children; never restart the tree.
- COMPILED-ONLY TRUTHS: Bun ignores `?query` on file: URL cache identity → plugin reload needs content-fingerprinted Blob module (loader.ts#importPluginModule); prove hot-reload on the compiled binary, never dev tree.

