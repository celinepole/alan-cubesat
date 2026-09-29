# ALAN Monitoring and Assessment System

An OML model of a 3U CubeSat mission that measures artificial light at night and delivers
radiance assessments to municipalities, observatories, astronomers and researchers.

The pages below are the authoring surface of the method. Work through them in order: each
one depends on facts established by the pages above it.

| # | Page | Writes to | Owner | Answers |
|---|---|---|---|---|
| 1 | [Component Decomposition](./ALAN%20CubeSat/Structure.md) | `structure/components` | Architect | Q1 |
| 2 | [Budget Allocations](./ALAN%20CubeSat/Allocations.md) | `structure/allocations` | Mission systems engineer | Q1 |
| 3 | [Part Budgets](./ALAN%20CubeSat/Part%20Budgets.md) | `structure/partbudgets` | Mass and power engineer | Q1 |
| 4 | [Interfaces](./ALAN%20CubeSat/Interfaces.md) | `interfaces/interfaces` | Subsystem interface owner | Q4 |
| 5 | [Connections and Flows](./ALAN%20CubeSat/Connections.md) | `interfaces/connections` | Interface owners jointly | Q4 |
| 6 | [Operating Modes and Contacts](./ALAN%20CubeSat/Operations.md) | `operations/modes` | Operations engineer | Q2 |
| 7 | [Requirements](./ALAN%20CubeSat/Requirements.md) | `requirements/requirements` | Requirements engineer | Q3 |
| 8 | [Verification Coverage](./ALAN%20CubeSat/Verification.md) | `requirements/verification` | V&V engineer | Q3 |

## Analysis

Authoring prescribes; these pages measure. They read the project bundle, because an
analysis scoped to one description reports compliance by seeing nothing to compare.

| Page | Answers | Views |
|---|---|---|
| [Analysis Dashboard](./ALAN%20CubeSat/Dashboard.md) | Q1, Q3, Q4 | chart, table, matrix, graph, tree |
| [Energy and Data Balance](./ALAN%20CubeSat/Energy%20Balance.md) | Q2 | table, scripted computation |
| [Conformance and Gap Audit](./ALAN%20CubeSat/Audit.md) | Does the model follow the method? | table, list |
| [The same audit, narrower scope](./ALAN%20CubeSat/Audit%20-%20Structure%20Scope.md) | What scope does to a finding list | table, list |

Opening any component also offers a **Component Analysis** view, which takes its subject
from the component rather than from a page.

What each question is answered by, and which gaps remain diagnosed rather than closed, is in
[ANALYSIS.md](../../../ANALYSIS.md).

Each page is a thin notebook: it names the description it writes to and composes a method
template. The prose, the editors, the shapes and the rules live in the method, under
`src/method/md/example.com/method/`. Change the method and every page changes with it.

What the method prescribes, and why, is in [METHOD.md](../../../METHOD.md) at the
repository root.
