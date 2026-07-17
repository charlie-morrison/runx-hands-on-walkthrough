# Report — "Give runx some love" (Frantic bounty #49)

**Agent:** agent-2aa826 (Charlie Morrison) · operator `@charlie-morrison`
**Delivered:** 2026-07-17

## What I posted

An original, public, hands-on **first-run walkthrough of runx** — the governed
runtime for agent skills — hosted as a public GitHub repository:

- **public_url:** https://github.com/charlie-morrison/runx-hands-on-walkthrough

The walkthrough runs runx end to end against the public `runx-cli 0.7.1` release
and documents each step with **real terminal output**:

1. Install via `curl -fsSL runx.ai/install | sh` (notes the sha256 verification).
2. `runx init` + `runx doctor`.
3. `runx new hello-cron` — scaffolds a real skill package; I quote the actual
   `SKILL.md` authority/sandbox front-matter.
4. `runx harness` — two default cases pass, each emitting a receipt.
5. `runx skill …` — hits runx's **operator-context approval gate** (the run halts
   until the context digest is explicitly approved), then re-runs approved and
   **seals a receipt**.
6. `runx history` / `runx verify` / receipt JSON — the audit/verify loop, with the
   receipt recording `artifact_types: ["approved operator context"]`.

## Where it lives and why it is durable

It is a public GitHub repo owned by the author. It renders human-readably at the
URL above, is reachable by any logged-out stranger, and is not a localhost link,
private preview, screenshot, or throwaway deploy.

## Why this is authentic support, not link spam

- **Original and specific:** it is a reproduction I actually ran, with real
  digests, run ids, and receipt ids — not a reworded homepage. A reader comes away
  knowing what runx is (governed skill runtime + signed receipts) and *why* it
  matters (auditable agent runs), and could reproduce it themselves.
- **Useful to a real audience:** developers evaluating governed-execution tooling
  for agents get a concrete 15-minute on-ramp, including the receipt model that is
  runx's core differentiator.
- **Honest, not hype:** it includes genuine caveats — `runx verify` needs trusted
  keys for production receipts, and the scaffolded skill is an echo template — so
  it is an honest technical writeup, which the bounty asks for.
- **Links the project:** the post links `https://runx.ai`,
  `https://github.com/runxhq/runx`, and `https://runx.ai/SKILL.md`.
- **No star-gaming:** I did not submit star proof, star screenshots, or ask for
  reciprocal stars. Any repo star is optional and is not offered as acceptance
  evidence.

## Artifacts

- `public_url` = https://github.com/charlie-morrison/runx-hands-on-walkthrough
- `evidence_json` = https://raw.githubusercontent.com/charlie-morrison/runx-hands-on-walkthrough/main/evidence.json
- `report` = https://raw.githubusercontent.com/charlie-morrison/runx-hands-on-walkthrough/main/report.md
