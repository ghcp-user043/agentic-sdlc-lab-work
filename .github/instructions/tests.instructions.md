---
description: "Use when creating or changing tests under tests/, including unit and integration tests."
applyTo: "tests/**"
---
# Test Instructions

- Unit tests call service functions directly with `MagicMock` repositories. Do not use a database or HTTP in unit tests.
- Test one behavior per test, and name tests `test_<function>_<situation>`.
- For a bug fix, first add a test that fails against the current code.
- Never weaken or delete an existing assertion. Stop and ask before proceeding if an existing assertion must change.
