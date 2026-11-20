You're helping a university student with the SE3002 Software Quality Engineering Final Project (Fall 2026). The full brief is attached. Read it fully before answering. It is graded strictly, with a viva where every member must explain their own area, so explain things in a way I can understand and defend, and don't just hand me finished work.

## Project in one line
Automated testing of an open-source MIS: functional tests, static analysis, mutation, CI gates, usability/accessibility, security, performance and recovery, plus test management documents. Groups of 2-3, 120 marks.

## Where we are
Environment: Windows, Python 3.12.10 as the only Python installed (I deliberately don't use venv, so packages go straight into this 3.12). Git is set up.

Repo selection is DONE and registered in the class sheet.
- Upstream: https://github.com/McShoothy/scoreboard (MIT licence)
- Our fork: https://github.com/CapsuleFlyest/scoreboard (my repo link is below; teammates are added as collaborators)
- Baseline commit (tagged `baseline` and pushed): 9c276da06a16356ce4be330a0b990dbddd4873f8
- App: "RobotUprising Tournament Scoreboard", a Flask + SQLite + Flask-SocketIO tournament manager. Admins create tournaments (single/double elimination, round robin, Swiss), teams register with a tournament code, admins confirm them, then the tournament starts and brackets are generated.
- Environment verified: all 50 upstream tests pass locally (`python -m pytest -q`). Upstream coverage was about 58% overall and only about 20% on register.py. Running the upstream tests modifies the tracked file `bracket_test_output.json`; it should be reset, not committed.
- Done on the fork: Issues and Actions turned on, collaborators added. Next commit: `scripts/seed_admin.py`.

## Chosen scope
Module: Team Registration and Management. Two workflows:
1. Public team registration (/register/ routes in app/blueprints/register.py).
2. Admin registration management: confirm / reject / check-in / manual add (routes in app/blueprints/admin.py under /admin/tournament/<id>/...).
Shared record: the tournament's registered-team count against max_teams (also registration_open / team is_confirmed).
Out of scope (to be listed in the test plan): brackets and bracket generation, TV display, iPad scoring, Socket.IO events, the external token API's scoring endpoints. I deliberately kept "start tournament" out because it pulls in bracket generation for five formats and would make 100% coverage unrealistic. My reading is that the brief's "100% statement and branch coverage of server-side business logic" means the logic behind the two selected workflows, not the whole app. This is an assumption I want to confirm with the instructor or TA.

## Draft test basis (verify every item against the code)
Requirements:
- FR1 valid code opens registration page; empty or invalid code shows an error
- FR2 valid registration creates a team; confirmed immediately unless the tournament requires confirmation
- FR3 registration refused when closed, started, past deadline, or full
- FR4 refused for a duplicate team name in the same tournament
- FR5 player names/email required only when the tournament requires them
- FR6 anonymous visitor cannot reach the admin registrations page
- FR7 admin can confirm a pending team
- FR8 admin can reject a team (deletes it, frees a slot)
- FR9 admin can toggle check-in
- FR10 admin can add a team manually, refused when full or duplicate

Business rules:
1. Boundary: team name 1-20 characters (public form); registration refused when the count reaches max_teams
2. Decision (multi-condition): Tournament.can_register() requires registration_open AND status == 'registration' AND deadline not passed AND registered count < max_teams
3. State change: is_confirmed = not require_confirmation on creation; pending -> confirmed by admin; check-in toggles afterwards; tournament status registration -> in_progress on start
4. Conditional fields: require_player_names makes both names required (max 30 each); require_email makes email required

Roles: anonymous registrant (can register, cannot use /admin) and admin (logged in, session-based). There is only one admin role in the DB; all admins are equal. We must record this as a justified assumption.

Assumptions to record in the test basis: duplicate names should be case-insensitive (admin add does this, the public form does not); the cap counts pending teams too; "reject" means delete.

Setup commands:
- Install: python -m pip install -r requirements.txt (drop gevent, gevent-websocket, eventlet, gunicorn lines if they fail; they are only for deployment)
- Start: python -m flask --app run run  (http://localhost:5000; `python run.py` crashes unless FLASK_DEBUG=true)
- Seed: python scripts/seed_admin.py (sets admin password to `password`)
- Reset: delete tournament.db, rerun the seed script

## Findings from exploring the app (verified by running it end to end over HTTP on Linux; re-verify on our machine)
1. README default login admin/password does NOT work: the hardcoded hash in app/__init__.py does not match `password` (also tried admin, admin123, changeme). We reseed with set_password.
2. Every 404 returns a 500: app/templates/404.html uses url_for('ui.index'), which does not exist (should be main.index). Confirming a nonexistent team (db.get_or_404) therefore returns 500.
3. Inconsistent rules: public registration allows names up to 20 chars with a case-sensitive duplicate check ("Alpha" and "alpha" both registered); admin manual add runs sanitize_team_name, which truncates to 8 chars, and compares case-insensitively.
4. Thin state machine: reject deletes the team (no rejected state); confirming an already confirmed team returns 200; check-in is a toggle.
5. From reading the code, not executed: max_teams is nullable but can_register() does `count >= self.max_teams`, which would raise TypeError if None.
6. An anonymous POST to a confirm endpoint returned 400 from the CSRF check, not a login redirect or 401. For the security access matrix, test with a valid CSRF token to observe the real login_required behaviour.
7. pip-audit on requirements.txt flagged known vulnerabilities in pinned Flask 3.0.0, Werkzeug 3.0.1, python-socketio 5.10.0, python-engineio 4.8.1 and python-dotenv 1.0.0.
8. Seen in code: Socket.IO uses cors_allowed_origins="*"; the internal api and input blueprints are CSRF-exempt; forms need CSRF tokens, so requests-based tests must fetch them first.
These are good candidates for the defect log (GitHub Issues), root cause analysis, risk register and security section. Please re-confirm each before we log it.

## What is NOT done yet
Everything from the brief beyond scope and setup: SonarQube and hand-calculated cyclomatic complexity, test plan, three-point estimates, risk register (8 product + 2 project risks, 5x5), exit criteria (3, measurable), traceability matrix, defect log with RCA (5 Whys + fishbone) and Pareto, OpenAPI spec and test-data generator (EP + 3-point BVA), unit tests with 100% statement/branch on scoped logic plus MCC and MC/DC on one critical function (likely can_register), API/integration tests, Playwright use-case tests (2 per member, each with 2 alternative and 2 exception flows), mutation testing (mutmut) with 2 custom operators and 15 surviving mutants analysed, GitHub Actions pipeline with blocking gates and one blocked PR, Lighthouse + axe-core before/after, security (ZAP passive, access-matrix checks, dependency audit), k6 performance runs and a failure-and-recovery test, weekly progress reports, completion report, AI-use record, and the final PDF + zip.
Our own tests go in tests/team/ (upstream tests earn no marks).

## Next steps, in order
1. Commit scripts/seed_admin.py and docs/test-basis.md (using the draft above after we verify it against the code).
2. Decide area ownership in the group: (1) static analysis, whitebox, mutation, CI; (2) API testing and data generation; (3) use cases, usability, accessibility, security, performance. Test management is shared. Ask me which area(s) I own.
3. Start the AI-use record immediately (tool, purpose, output, what I checked or changed) and keep prompts. Academic integrity is graded strictly and undisclosed AI use is penalised.
4. Run SonarQube, write the risk register and test plan, then build the generator and tests per area.

## How I want you to work
- I'm a student in a hurry; keep answers practical and concise, and tell me what to run in PowerShell on Windows.
- Make me understand what's being done, since I'll be examined on my area in a live viva (rerun a test, explain assertions, handle a live change).
- Don't invent evidence, results, or screenshots; if you can't verify something, say so.
- Ask me which area I own before diving deep, then start with the next step above.

My repo: https://github.com/CapsuleFlyest/scoreboard