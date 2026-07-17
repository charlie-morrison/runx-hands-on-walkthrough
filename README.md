# runx: a hands-on first-run walkthrough (from an automation agent's seat)

A short, reproducible walkthrough of **[runx](https://runx.ai)** — *the governed
runtime for agent skills* ([github.com/runxhq/runx](https://github.com/runxhq/runx),
MIT, built in Rust). Every command and every block of output below was run on a
plain Linux box against the public release **`runx-cli 0.7.1`**. Nothing is
mocked; if you run the same commands you should see the same shapes.

I build cron/automation agents, so I looked at runx through one question: *when
an agent runs a skill that could touch money, deploys, or prod, what proof do I
have afterward about what it actually did?* runx's answer is **scoped authority +
a signed receipt for every run**. This walkthrough shows that loop end to end.

## What runx is (in one paragraph)

A skill is a `SKILL.md` file that packages a unit of expertise — its inputs, the
command that does the work, and a sandbox/authority policy. You point any agent
(Claude Code, Cursor, Codex, GPT, Gemini, Aider) or the `runx` CLI at a skill
reference and it runs under a declared policy, then **seals a receipt** you can
inspect, verify, and replay. The pitch on the tin — *"your agent, off the leash
but on the record"* — turns out to be literal: the run below refused to proceed
until I explicitly approved its operator context.

## 1. Install (prebuilt binary, checksum-verified)

```
$ curl -fsSL runx.ai/install | sh
runx: resolving latest release...
runx: downloading runx 0.7.1 (x86_64-unknown-linux-musl)
runx: checksum verified
runx: installed to .../bin/runx
$ runx --version
runx-cli 0.7.1
```

The installer detects your OS/arch, pulls the matching release archive from
GitHub, and **verifies its `sha256` before installing** — worth noting if you're
security-conscious about `curl | sh`. It honors `RUNX_INSTALL_DIR`, so I dropped
the binary in a project-local dir instead of `$HOME/.local/bin`.

## 2. Initialize a project and health-check it

```
$ runx init
  runx project init  success
  project     .../demo/.runx
  project_id  proj_5ab1bda1-b9d5-426a-9554-7c3e9aec3401

$ runx doctor
  ✓  doctor  0 error(s), 0 warning(s)
```

`runx doctor` is the "is my environment sane" command — it also takes
`path | authority | registry` subtargets when you want to inspect one facet.

## 3. Scaffold a skill

```
$ runx new hello-cron
  runx new  success
  skill      hello-cron
  files      5
```

That gives you a real, runnable skill package:

```
hello-cron/SKILL.md    # front-matter: inputs, source command, sandbox policy
hello-cron/run.mjs     # the actual work (here: echo the input back)
hello-cron/X.yaml      # harness cases that lock the behaviour
hello-cron/README.md
hello-cron/.gitignore
```

The interesting part is the `SKILL.md` front-matter — the policy travels *with*
the skill:

```yaml
source:
  type: cli-tool
  command: node
  args: [run.mjs]
  timeout_seconds: 30
  sandbox:
    profile: readonly          # <- authority is declared, not assumed
    cwd_policy: skill-directory
inputs:
  message:
    type: string
    required: true
```

## 4. Lock behaviour with the harness

```
$ runx harness hello-cron
{
  "status": "passed",
  "case_count": 2,
  "case_names": ["hello-cron-smoke", "hello-cron-empty-message-fails"],
  "receipt_ids": ["sha256:e1ebb629...", "sha256:f57b3581..."]
}
```

Two cases ship by default — a smoke test and a negative case — and **each one
emits a receipt**. The harness is how you keep a skill's contract from drifting
as an agent (or a human) edits it.

## 5. Run the skill — and meet the governance gate

This is the moment that made runx click for me. A first `runx skill` invocation
does **not** just run:

```
$ runx skill hello-cron --input message="shipped by a cron agent"
Prepared run
  Skill:  hello-cron
  Digest: sha256:6a20c537...
Approval required
Rerun the same command with:
  --approve-operator-context sha256:6a20c537...
```

It computed a digest of the operator context and **halted for explicit
approval**. You re-run with the digest to consent:

```
$ runx skill hello-cron --input message="shipped by a cron agent" \
    --approve-operator-context sha256:6a20c537...
status: sealed
skill: hello-cron
run_id: run_default_0b46228e9b9e
receipt_id: sha256:982584a6687a7c7cec4be1233b5e69fe91bca705c596d78d290d190e64e17250
summary: cli-tool default completed
```

For an unattended cron agent this is exactly the seam you want: the authority a
run will exercise is pinned to a digest you can whitelist, instead of the agent
silently deciding for itself.

## 6. The receipt: history, verify, inspect

Every run lands in a local receipt store:

```
$ runx history
  history  3 receipt(s)
  closed  hrn_run_default_0b46228e9b9e_default  sha256:98258...
  failed  hrn_run_default_15c53b41755b_default  sha256:f57b3...
  closed  hrn_run_default_9b2d43affbf4_default  sha256:e1ebb...
```

```
$ runx verify sha256:982584a6... --allow-local-development-signatures
signature mode: local-development
tree sha256:982584a6... (1 receipt): ok
verification: ok
```

The receipt itself is structured and content-addressed:

```json
{
  "id": "sha256:982584a6687a7c7cec4be1233b5e69fe91bca705c596d78d290d190e64e17250",
  "receipt_ref": "runx:receipt:sha256:982584a6...",
  "status": "closed",
  "created_at": "2026-07-17T12:07:20.854Z",
  "summary": "cli-tool default completed",
  "actors": ["runx:principal:local_runtime"],
  "artifact_types": ["approved operator context"]
}
```

Note `artifact_types: ["approved operator context"]` — the receipt records
*that* the governance gate from step 5 was cleared. `runx history <id> --json`
gives you the full object for auditing or replay.

## Honest notes (the bar here is honesty, not hype)

- **`runx verify` is strict by design.** Verifying a production receipt needs
  trusted keys (`RUNX_RECEIPT_VERIFY_KID` + an ed25519 public key); local runs
  require `--allow-local-development-signatures`. Good default — it won't silently
  "verify" an unsigned receipt — but expect the extra step.
- **The default `hello-cron` skill is an echo.** The scaffold is a template; the
  value shows up once you replace `run.mjs` with real work and add harness cases.
- **The approval gate is per operator-context digest**, so once you've approved a
  stable context you can whitelist that digest in automation rather than approving
  interactively each time.

## Why this matters

Most "agent runs a tool" stories give you a log line and a shrug. runx gives you
a **receipt**: a content-addressed, signable record of what ran, under what
authority, with what result — the same primitive whether you're on Claude Code or
a headless cron. If you care about agents that can be pointed at consequential
systems *and* audited afterward, it's worth 15 minutes.

- Project: https://runx.ai
- Source (MIT, Rust): https://github.com/runxhq/runx
- Spec your agent can read directly: https://runx.ai/SKILL.md

*Reproduced against `runx-cli 0.7.1`, 2026-07-17. Walkthrough by
[@charlie-morrison](https://github.com/charlie-morrison).*
