# Testing and Quality

Read this when writing, changing, or reviewing tests, or when finishing a feature. Our stance: **fewer, higher-value tests, but the right ones, written as you build.** Coverage is not optional, but coverage of the wrong things is waste.

---

## Philosophy

Test the feature, not just the code. Type checking and test suites verify correctness at the code level; they don't tell you the feature works as intended. Run the thing and use it before calling it done.

Two failure modes, both bad:

- **Under-testing:** shipping a feature with no tests, so the next change silently breaks it.
- **Over-testing:** a suite full of brittle, low-value tests that break on every refactor, test framework internals, or assert trivia. These rot, get ignored, and eventually get deleted wholesale, taking the good tests with them.

We aim for the middle: a small suite of tests that each earn their keep by catching a real class of regression.

---

## What to Test (high value)

Write a test when a failure there would actually hurt:

- **Business logic and rules.** Scoring, pricing, permissions, state machines, anything with branching logic or edge cases. This is the highest-value target.
- **Data transformations.** Parsers, importers, mappers, serializers. Easy to test, high regression risk.
- **API contracts.** Endpoint inputs/outputs, validation, error responses, auth boundaries. The contract other code depends on.
- **Bug fixes.** When you fix a bug, add the test that would have caught it. This is the single most cost-effective test you can write, and it's mandatory, not optional.
- **Anything you're about to refactor.** Pin behavior with a test *before* you change the internals, so the refactor is provably safe.

## What Not to Test (low value, avoid)

Skip these unless there's a specific reason. They cost more to maintain than they catch:

- **Framework or library internals.** Don't test that React renders or that the ORM saves a row. Trust your dependencies.
- **Trivial getters/setters and pass-throughs.** No logic, nothing to break.
- **Implementation detail instead of behavior.** Asserting that a private method was called, or snapshotting a giant DOM tree, breaks on every harmless refactor. Test observable behavior and public contracts.
- **Over-mocked tests** where the mocks are so extensive the test only verifies the mocks. If you mock everything, you've tested nothing.
- **Generated or boilerplate code** with no custom logic.

If you can't name the regression a test prevents, don't write it.

---

## Coverage Expectations (enforced)

Coverage is a floor on diligence, not a vanity metric. The rule is about *what changed*, not a global percentage:

- **New feature work ships with tests for its business logic and contracts.** A feature PR with zero tests on non-trivial logic is incomplete. Say so in review.
- **Touching older code means leaving it tested.** If you modify a function that has no tests and it has real logic, add the tests for the path you touched before or alongside your change. Don't expand untested surface area.
- **Every bug fix gets a regression test.** No exceptions, this is the cheapest, highest-value test there is.
- **Don't chase a coverage number by testing trivia.** Hitting 90% by testing getters is worse than 60% that covers the logic that matters. Coverage tools measure lines hit, not value, read them with judgment.

In review, the question is not "is coverage high?" but "is the logic that could break actually covered, and are these the right tests?"

## Periodic Test Review

Tests are code, and they rot. Re-examine the suite periodically (when working in an area, during a refactor, or as a deliberate pass):

- **Delete tests that no longer earn their keep** — testing removed behavior, duplicating another test, or so brittle they're muted/skipped more than they pass.
- **Fix or remove flaky tests.** A test that fails randomly trains everyone to ignore failures. A flaky test is worse than no test.
- **Collapse redundancy.** If five tests exercise the same path, keep the clearest one.
- **Check that the important paths are still covered** after features change shape. Coverage drifts as code evolves.

A lean suite that people trust beats a large suite people ignore.

---

## Approach

- **Golden path first.** Make sure the intended workflow works end to end.
- **Edge cases second.** Empty states, error states, boundary values.
- **Regression awareness.** When changing shared code, check that existing features still work.
- **Browser-based verification** for all UI work. Use automated browser tools (Playwright, Puppeteer) for repeatable testing.
- **Backend tests** for API logic and business rules (RSpec for Rails, or equivalent).

---

## Smoke Testing

A **smoke test** is the fast, shallow "does it boot and do the main thing" check you run after a change, before deeper testing or before calling work done. It is not the full suite and not exhaustive edge-case testing, it's the minimum proof that you didn't break the obvious.

Run a smoke test after any non-trivial change. The smoke test for a project answers: *does the app start, and does its primary golden path still work?*

- **App boots.** Dev server starts, build succeeds, no startup crash, no console errors on load.
- **Primary golden path works.** The one or two flows the app exists for. For an API: the main endpoints return expected shapes. For a UI: the core user action completes (load the page, do the main thing, see the result). For a CLI: the main command runs and produces output.
- **Nothing obviously regressed.** The feature you just changed works when you actually use it, in the browser/terminal, not just in a green test run.

Keep it fast (seconds to a couple of minutes) and runnable on demand. Where it's worth it, encode the smoke path as a short automated script or a tagged subset of the test suite (`@smoke`) so it's repeatable. Document the project's smoke check in its `CLAUDE.md` so anyone (human or agent) can run it.

A passing test suite is not a substitute for a smoke test, and a smoke test is not a substitute for the suite. The suite proves the code is correct in the ways you anticipated; the smoke test proves the assembled thing actually runs.

---

## Security Audits

- Track dependencies with automated tools (Dependabot, `npm audit`, `bundle audit`)
- Pre-commit hooks for catching common issues
- Treat any secret in version control as a security incident (see `principles.md` → Security and global CLAUDE.md)
