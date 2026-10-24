# CLAUDE.md - rules for Claude Code in this repo

Project: [ PROJECT DESCRIPTION WRITTEN HERE ]

Read every session, in order: CHECKPOINT.md (orientation, in-flight state, and the index of older decisions still in force); DECISIONS.md from D235 to the end (authoritative; overrides everything), and an older entry only when a current one cites it (search by its D-number); the Status section of results/SUMMARY.md, and its run rows only when needed (search). HANDOFF.md is historical - §0 working rules only.

You execute; planning happens in a separate chat. Don't change scope, add datasets or methods, or edit HANDOFF.md / DECISIONS.md. Propose changes in your report.

1. Verify before building. If a step rests on an unverified assumption, check it first or say so.
2. Run it and see. A small script and a real run beat discussion.
3. Numbers come only from code. Every reported number is written into results/ with commit hash, exact command, data/split and checkpoint. Never type a number by hand.
4. Environment: never sudo. Never touch GPU drivers, system CUDA or system Python. Installs go into named conda envs; exact versions go in results/ENV.md.
5. Git: never commit data/, checkpoints/, outputs/, third_party/. Only small files in results/. Commit often; push after each phase.
6. Long jobs (downloads, inference, training) run detached with setsid and nohup, never as a child of the session, with logs in outputs/logs/ and the PID recorded in the phase notes. A session ending must not be able to kill them.
7. Stop and report, don't push on, when: an environment problem passes 2 hours; anything needs sudo or a system change; a sanity check fails; a reproduction misses its published number by more than {TOLERANCE}.
8. Every metric or evaluation function gets a synthetic test with known answers ({TEST_CASES}) in tests/ before it touches real data.
9. Apart from conda envs, write only inside this repo.
10. End every task with a report: commands run, numbers (with results/ paths), failures, what's still unverified. Rewrite the Status section of results/SUMMARY.md (max 5 lines) and append run rows.
11. Context budget. Keep tool output short: tail logs, select JSON keys, run tests quietly and print only failures and totals, never print a results file in full. Jobs longer than about 15 minutes run in the background; record their PIDs and log paths in the phase notes file and end the turn instead of watching them. Delegate large-file reading and log scanning to a subagent where possible. Keep outputs/<phase>/notes.md current so a fresh session can resume.
12. Agents. Use subagents for reading large files and scanning logs (rule 11), not as default reviewers. At most two review agents per session, only for code that computes a pre-registered verdict or a claim; they read only what their review needs.
13. Verification in proportion. Close-outs recompute independently what is claimed, pre-registered, or quoted in the final report (D237(a)(viii)); every other number is checked against its source by hash or by re-running the code that wrote it. Reuse the existing verifier modules (scripts/{VERIFIER}_*) instead of writing new ones per phase.