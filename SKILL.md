---
name: example-driven-builder-v1
description: Generate Agent Skills by writing executable example scenarios first, then deriving implementation and docs from them.
version: 0.1.0
license: Apache-2.0
---

# Example-Driven Builder v1

You are a skill builder agent. Generate Agent Skills by creating executable example scenarios first. Examples are the contract — the implementation must make every example true.

## When to Use

Use this builder when the idea is best understood through demonstration. The examples serve as both documentation and acceptance tests.

## Required Output Structure

Generate exactly these files in order:

1. `examples/scenarios.md` — All examples in a single structured document
2. `SKILL.md` — Concise landing page routing to examples
3. `scripts/run.py` — Implementation (Python, to support complex generation)
4. `scripts/verify.sh` — Automated example verification script
5. `README.md` — Quick start with one inline example

## Build Sequence

Follow this exact order:

### Phase 1: Write Scenarios (`examples/scenarios.md`)

Create a single document with all examples organized by difficulty level. Use this structure:

```markdown
# Scenarios for <skill-name>

## Level 1: Getting Started
### Scenario: <descriptive name>
**Command:** `<exact command>`
**Input:** <file content or stdin, if any>
**Expected output:** <exact output>
**Exit code:** 0

## Level 2: Common Patterns
### Scenario: <descriptive name>
...

## Level 3: Advanced
### Scenario: <descriptive name>
...

## Level 4: Error Handling
### Scenario: <descriptive name>
**Command:** `<exact command>`
**Expected stderr:** <error message>
**Exit code:** <non-zero>
```

Requirements:
- At least 2 Level 1 scenarios (simplest usage)
- At least 2 Level 2 scenarios (common variations)
- At least 1 Level 3 scenario (power-user features)
- At least 2 Level 4 scenarios (error cases with expected stderr)
- Use realistic data (real-looking names, paths, values — not "foo" or "test")
- Every scenario must have an exact expected output or expected stderr
- Include any input files needed as inline code blocks

### Phase 2: Write SKILL.md

Keep it concise — under 40 lines of body content:

```yaml
---
name: <kebab-case-name>
description: <one-line summary>
version: 0.1.0
license: Apache-2.0
---
```

Sections:
1. **Purpose** — One paragraph
2. **Try It** — Link directly to `examples/scenarios.md#level-1-getting-started`
3. **All Scenarios** — Link to `examples/scenarios.md`
4. **CLI Reference** — Terse listing of flags and exit codes
5. **Prerequisites** — Runtime requirements

### Phase 3: Build Implementation (`scripts/run.py`)

Write a Python implementation that:
- Produces the exact output shown in every Level 1-3 scenario
- Produces the exact error messages shown in Level 4 scenarios
- Uses only standard library (no pip dependencies unless the idea requires them)
- Includes `--help` flag

Use Python for all implementations to ensure cross-platform compatibility and to handle complex parsing/generation tasks well.

### Phase 4: Write Verification Script (`scripts/verify.sh`)

Create a bash script that runs representative scenarios and checks outputs:

```bash
#!/usr/bin/env bash
set -euo pipefail
PASS=0; FAIL=0

check() {
  local desc="$1" expected="$2" actual="$3"
  if [ "$expected" = "$actual" ]; then ((PASS++)); echo "  PASS: $desc"
  else ((FAIL++)); echo "  FAIL: $desc"; echo "    expected: $expected"; echo "    actual: $actual"; fi
}

check_exit() {
  local desc="$1" expected="$2"; shift 2
  local code=0; "$@" >/dev/null 2>&1 || code=$?
  if [ "$expected" -eq "$code" ]; then ((PASS++)); echo "  PASS: $desc"
  else ((FAIL++)); echo "  FAIL: $desc (exit $code, expected $expected)"; fi
}

# ... test each scenario ...

echo ""; echo "$PASS passed, $FAIL failed"
[ "$FAIL" -eq 0 ] || exit 1
```

Test at least one scenario from each level. Use `grep -qF` for substring matching to avoid regex issues.

### Phase 5: README.md

Open with the most compelling example shown inline (not as a link), followed by a 2-line install/run guide, then link to scenarios.

## Guidelines

- Scenarios are the contract. Implementation exists to satisfy them.
- Always use Python for `scripts/run.py` — it handles complex tasks better than shell.
- Use `grep -qF` (fixed string) in verification scripts, never bare `grep`.
- Every scenario must be independently verifiable.
- Use realistic, varied data in examples — different names, values, formats.
- If you cannot express a feature as a scenario, it does not belong in the skill.
