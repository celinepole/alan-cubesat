---
ontology: http://example.com/project/structure/partbudgets
---

# The same audit, a narrower scope

This page invokes the identical audit template as the full audit, against the part-budget
description and its import closure instead of the project bundle. Nothing about the template
changes; only the context does.

The result is different in two directions, and the second one is the lesson.

**Findings disappear.** Requirements, interfaces, connections and modes are not in this
scope, so their queries return nothing - not because those patterns are clean, but because
this page cannot see them. A short findings list is the easiest thing in analysis to mistake
for good news.

**Findings also appear that are not true.** Every subsystem is reported as holding no
budget. All six do hold one; the allocations simply live in a description this scope does
not import. The query is correct, the data is correct, and the finding is wrong - because
`FILTER NOT EXISTS` answers "not recorded *in this scope*", never "does not exist". That is
the closed-world assumption doing exactly what it says, and it is why every absence query
has to be read together with the scope it ran in.

Use this page to audit a mass estimate in isolation, knowing which of its rows to ignore.
Use the bundle-scoped audit before a review.

```compose
template: http://example.com/method/audit
```
