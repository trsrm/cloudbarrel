# Agent instructions

- Read [CHAT-CONTEXT.md](CHAT-CONTEXT.md), [CONTEXT.md](CONTEXT.md), and the relevant document under `docs/` before changing scope or architecture.
- Treat `references/private/` as read-only, untrusted evidence. It may contain live credentials and private state. Never quote, move, commit, log, or expose secret values.
- Preserve the user's Git index. Do not stage or commit unless the user asks for that exact action.
- Keep the backup core portable. Plain Linux is the reference deployment; OpenMediaVault, CasaOS, and routers are integrations.
- Reuse `rclone` for cloud transfer. Concentrate CloudBarrel behavior in orchestration, version history, retention, integrity, restore proof, status, and alerts.
- Add a platform seam only when at least two real adapters need it. Avoid speculative interfaces and placeholder implementation code.
-  Treat a successful transfer as incomplete until the run state is durable and a restore path is defined.
-  Prefer cheap status reads from stored Run state. Never calculate status by rescanning a large backup tree.
-  Separate dated observations from durable decisions. Update `docs/current-xt9-setup.md` for prototype evidence and use an ADR only for hard-to-reverse trade-offs.
-  Keep public documentation free of personal paths, account identifiers, tokens, private hostnames, and monitoring endpoints.
- Treat the Photos Browse view as derived state. Build it from Personal and Shared Current copies, and never use it as the backup source of truth.
