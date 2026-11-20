# AI-Use Record

Course: SE3002 Software Quality Engineering, Final Project (Fall 2026)
Rule (brief section 7): AI is allowed only if every output is checked and disclosed here (tool, purpose, output, what was checked or changed). Prompts are kept in `docs/ai-prompts/`.

Rules for this file
- One row per AI session or task. Add a row the same day, not at the end.
- "Checked / changed" is written by the member who used the output, in their own words. Say what you ran or read yourself and what you rejected or edited. Leave it as TODO until it is true.
- Do not edit or delete old rows. Add a correction row instead.

| # | Date | Member | Tool | Purpose | Output received | Checked / changed by the member | Prompts file |
|---|---|---|---|---|---|---|---|
| 1 | 2026-11-19 | Yap | Claude (Sonnet 5.5) via Claude Code | Verify the draft test basis against the source code and run the app's registration behaviour to confirm suspected defects | `docs/test-basis.md` draft: verified requirements FR1-FR12, business rules, roles, findings F1-F8 and new findings N1-N4; pip-audit output (14 unique advisories) | TODO: record what you re-ran or re-read yourself (for example: re-ran the probes on Windows, opened `models.py` `can_register`, confirmed the 404 template line, edited or removed any row you could not explain) | `docs/ai-prompts/` |
| 2 | 2026-11-20 | Yap | Claude (Sonnet 5.5) via Claude Code | Explain the project brief components and start-order; draft this AI-use record template | Roadmap of components and marks; this template; SonarQube setup steps | TODO | `docs/ai-prompts/` |

## Things the AI output could not verify (so the member must)
- Windows install, start and reset commands.
- The `baseline` tag on the fork.
- Any SonarQube, Lighthouse, ZAP or k6 result. None has been produced yet, so none may be cited until a member runs the tool.
