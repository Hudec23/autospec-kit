# Planning step 4 — Audit as the executor

Pretend you are the executor: a 27B model, 64K context, that sees ONE task file plus the files it
lists, nothing else, and cannot ask anything. Go through every task in `plan/tasks/` in
dependency order and, for each, answer:

1. Could I start typing code within one minute of reading this? If I would need to search the
   repo to find a type, path, import or convention — the task is missing Context.
2. Is any word vague ("appropriate", "handle errors", "as needed", "etc.", "similar to")? Replace
   it with the concrete thing.
3. **Test tasks:** could I satisfy every listed test case with weak tests (only checking "no
   exception", only the status code, mocking the thing under test)? Make each case state the exact
   expected value. Could a correct implementation still fail them (over-specified internals,
   ordering, timestamps)? Loosen to the public behaviour. Are the needed fakes/fixtures in the repo?
   Does the stub exist so the tests compile? A stub that raises at once hides bugs in the test's
   own setup, assertions and teardown (unhashable values, cleanup order, leaked connections): read
   those parts as if the implementation were already correct.
4. **Impl tasks:** could I pass the locked tests with a wrong implementation (hard-coded return,
   skipped validation)? Add test cases to the paired test task until not. Does the feature's last
   task lock and run its acceptance tests?
5. Will I need a library, env var, migration, generated file or fixture that doesn't exist yet
   and isn't created by an earlier task? Add it to the skeleton or an earlier task.
6. Does the task fit: ≤3 edit files, ≤15K tokens to read, one concern? Count what the executor will
   *want* to open, not only `read:`: a task covering many stages/endpoints/schemas makes it read
   every contract file. Point it to example fixtures, inline the exact values, or split it.
7. Does the Do-not list cover the obvious traps (editing tests, changing contracts, adding deps,
   renaming exported symbols other tasks depend on)?

Then, once for the whole plan:

8. Bring up the dev/test compose and check `docker compose ps`: every infra container is
   `healthy`, none `Exited`. Run every service's `baseline_tests`.
9. `python3 scripts/autospec-run.py --dry-run` parses every task and lists them all; the first
   tasks have no unmet `depends_on`.

Fix every finding directly in the files, then re-run the checker (step 3D).

Output a table: task id | findings | fixed (y/n). End with "READY FOR UNATTENDED RUN: yes/no" — yes only with
`PLAN CHECK: 0 errors`.
