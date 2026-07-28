## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/150

**Issue title:** Tech detector counts vendored and build-output files, skewing language detection

**Tier:** [x] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem Summary:**

The TechDetector tool in agent/tools/tech_detector.py is supposed to
ignore files inside vendor/build folders like node_modules/ and build/
when detecting a repository's primary programming language. However,
its skip-list checks for patterns like "/node_modules/" with a leading
slash, while file paths passed into the tool (e.g. "node_modules/lib/index.js")
often don't have that leading slash. Because of this mismatch, those
files are never filtered out and get counted as if they were part of
the actual codebase. As a result, a repo that is mostly Python can be
incorrectly reported as "primarily JavaScript" just because it has a
few bundled JS dependency files. Fixing this means correcting the skip
logic so it matches these paths regardless of a leading slash, restoring
accurate language detection.

**Branch name:** fix/150-tech-detector-vendored-files

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

**Selection notes ("Is this right for me?" checklist):**

I chose Tier 1 issue #150 over #153 (also Tier 1) because it involves a
single, well-contained bug in one file (agent/tools/tech_detector.py),
has clear reproduction steps and named failing tests, and touches the
agent/ area of the codebase, which interests me. I avoided #153 for a
different reason: it had an unusually high number of claims and several
already-open pull requests, and I wanted to reduce the temptation to
reference existing solutions before reasoning through the bug myself.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/DataMnk/pathreview/commit/939b152

**Reproduction summary:**
I reproduced the bug locally using the Python REPL with the exact steps
from the issue: `TechDetector().execute({'files': [...]})` with a file
list containing `node_modules/lib/index.js` and `build/bundle.js` among
Python files. The result's `primary_language` came back as `JavaScript`
instead of the expected `Python`, confirming that `_should_skip_file()`
fails to filter out these vendored/build paths because it checks for
patterns with a leading slash (e.g. `"/node_modules/"`) that don't match
paths without one (e.g. `"node_modules/lib/index.js"`).

**PLAN.md link:** https://github.com/DataMnk/pathreview/blob/fix/150-tech-detector-vendored-files/PLAN.md


**Blockers or open questions:**
None major yet. I still need to confirm whether file paths can ever arrive
with backslashes instead of forward slashes on Windows, but I don't
expect this to block the fix in Week 9.