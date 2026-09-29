---
ontology: http://example.com/project/bundle
---

```compose
template: http://example.com/method/part-budgets
target: http://example.com/project/structure/partbudgets
```

## What this page currently shows

Two findings are live in the table above, and both are the reason this project exists.

**ADCS is over its allocation.** The two ADCS parts sum to 0.585 kg against an allocation
of 0.531 kg. The allocation is the subsystem total row of TrueSightSAT Table 14; the two
line items in the same table are the parts above. A 54 g discrepancy inside one table,
which neither the report review nor the SysML model caught, because neither related the
two figures in one artifact.

**The system power draw exceeds generation.** The leaf parts draw 21.91 W orbit-average
against the 18.44 W the solar array generates. The report reaches the same conclusion by a
different route in the same section, recording a per-orbit deficit of -8.0 Wh and
recommending a reduced imaging duty cycle. The static form of the problem is visible here;
the dynamic form is Q2, on the Operations page.

Neither finding makes the model inconsistent, and that is correct. An overrun is an
engineering finding about the design, not a contradiction in the knowledge.
