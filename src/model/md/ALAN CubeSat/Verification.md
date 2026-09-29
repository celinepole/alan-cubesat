---
ontology: http://example.com/project/requirements/verification
---

```compose
template: http://example.com/method/verification
```

## Coverage today

Three of seven requirements carry no activity: `SUB_ADCS_MASS_01`, `SUB_STR_MASS_01` and
`OP_MIS_010`. The first two are the subsystem mass constraints the part budgets already
show being broken, which makes them the least comfortable ones to leave uncovered.

They are left uncovered deliberately in this increment. Q3 asks which requirements are
supported only by assertion; if every requirement carried an activity, the answer would be
an empty list and would demonstrate nothing.
