---
template:
  id: http://example.com/method/energy-balance
  name: "Energy and Data Balance"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Energy and Data Balance

**Q2: what is the highest imaging duty cycle this spacecraft can sustain?**

This is the question the model has been carrying inputs for since Deliverable 3 without
answering. It cannot be answered by a query. A query can select the modes, their duty
cycles, their draw and the contact capacity, but the answer needs arithmetic over those
values and a solve for the quantity that makes the balance close. SPARQL selects the
subgraph; the script computes.

## The inputs, straight from the model

```table
---
orderBy: ["DutyCycle desc"]
---
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Mode ?DutyCycle ?PowerW ?DataRateMbps
WHERE {
  ?mode a alan:OperatingMode ;
        alan:dutyCycle ?DutyCycle ;
        alan:modePowerDraw ?PowerW .
  OPTIONAL { ?mode alan:modeDataRate ?DataRateMbps }
  BIND(REPLACE(STR(?mode), "^.*[#/]", "") AS ?Mode)
}
```

```table
PREFIX alan: <http://example.com/method/vocabulary#>
PREFIX oml:  <http://opencaesar.io/oml#>
PREFIX dc:   <http://purl.org/dc/elements/1.1/>

SELECT ?Contact ?DurationS ?RateMbps ?CapacityGb
WHERE {
  ?contact a alan:ContactOpportunity ;
           alan:contactDuration ?DurationS ;
           alan:downlinkRate ?RateMbps .
  BIND(ROUND(?DurationS * ?RateMbps / 10) / 100 AS ?CapacityGb)
  BIND(REPLACE(STR(?contact), "^.*[#/]", "") AS ?Contact)
}
```

## The computation

The script below fetches those rows, computes the orbit-average draw, and solves for the
imaging duty cycle at which generation and consumption balance. Four inputs it needs are
**not** model facts, and they are declared as parameters at the top rather than smuggled in:
orbit period, the generation figure, the passes available per day, and the assumption that
safe mode absorbs whatever time imaging does not use. Three of the four are candidates for
the vocabulary; the fourth is an operational policy.

```python
import json

# --- parameters that are NOT model facts -------------------------------
ORBIT_PERIOD_S   = 94.6 * 60    # 505 km SSO; altitude is prose, not a model fact
GENERATION_W     = 18.44        # SystemAllocation allocatedPower, restated here
PASSES_PER_DAY   = 4            # not modelled: one ContactOpportunity, no cadence
# -----------------------------------------------------------------------

MODES = """
PREFIX alan: <http://example.com/method/vocabulary#>
SELECT ?mode ?duty ?power ?rate WHERE {
  ?mode a alan:OperatingMode ;
        alan:dutyCycle ?duty ;
        alan:modePowerDraw ?power .
  OPTIONAL { ?mode alan:modeDataRate ?rate }
}
"""

CONTACT = """
PREFIX alan: <http://example.com/method/vocabulary#>
SELECT (SUM(?d * ?r) AS ?mbPerPass) WHERE {
  ?c a alan:ContactOpportunity ;
     alan:contactDuration ?d ;
     alan:downlinkRate ?r .
}
"""

def num(x):
    return float(str(x))

rows = await query(MODES)
modes = {}
for r in rows:
    name = str(r["mode"]).split("#")[-1]
    modes[name] = (num(r["duty"]), num(r["power"]),
                   num(r["rate"]) if r.get("rate") is not None else 0.0)

contact_rows = await query(CONTACT)
mb_per_pass = num(contact_rows[0]["mbPerPass"])

# 1. where the design sits today
draw = sum(duty * power for duty, power, _ in modes.values())
margin = GENERATION_W - draw

# 2. solve for the imaging duty cycle that closes the energy balance.
#    Imaging gives its time back to safe mode; downlink is fixed by the pass.
imaging_p = modes["ImagingMode"][1]
safe_p    = modes["SafeMode"][1]
down_d, down_p, _ = modes["DownlinkMode"]
slope     = imaging_p - safe_p
constant  = down_p * down_d + safe_p * (1 - down_d)
duty_energy = (GENERATION_W - constant) / slope

# 3. and for the duty cycle the downlink can drain
imaging_rate = modes["ImagingMode"][2]
capacity_mb_day = mb_per_pass * PASSES_PER_DAY
duty_data = capacity_mb_day / (86400 * imaging_rate) if imaging_rate else float("inf")

result = {
    "current_duty":  modes["ImagingMode"][0],
    "draw":          round(draw, 2),
    "generation":    GENERATION_W,
    "margin":        round(margin, 2),
    "duty_energy":   round(duty_energy, 3),
    "duty_data":     round(min(duty_data, 1.0), 3),
    "binding":       "energy" if duty_energy <= duty_data else "downlink",
    "capacity_gb":   round(capacity_mb_day / 1000, 1),
    "passes":        PASSES_PER_DAY,
}
store.set("balance", json.dumps(result))

print(f"orbit-average draw   {result['draw']} W against {GENERATION_W} W generated")
print(f"margin               {result['margin']} W")
print(f"sustainable imaging duty cycle:")
print(f"  limited by energy    {result['duty_energy']:.1%}")
print(f"  limited by downlink  {result['duty_data']:.1%}  at {PASSES_PER_DAY} passes/day")
print(f"currently asserted     {result['current_duty']:.1%}")
```

```javascript
const b = JSON.parse(store.get("balance"));
const limit = Math.min(b.duty_energy, b.duty_data);
const over  = b.current_duty > limit;

display(clientWidget(`
  <div style="font-family:system-ui;line-height:1.5">
    <div style="font-size:2rem;font-weight:700;color:${over ? "#DC2626" : "#10B981"}">
      ${(limit * 100).toFixed(1)}%
    </div>
    <div>highest sustainable imaging duty cycle, limited by <b>${b.binding}</b></div>
    <div style="margin-top:.5rem">
      asserted in the model: <b>${(b.current_duty * 100).toFixed(1)}%</b>
      &nbsp;·&nbsp; power margin: <b>${b.margin} W</b>
      &nbsp;·&nbsp; downlink: <b>${b.capacity_gb} Gb/day</b> over ${b.passes} passes
    </div>
  </div>
`));
```

## What the numbers say

**The asserted duty cycle is not sustainable on either constraint.** The model asserts
93.75 % imaging. Energy closes at about 63 %, and at four passes a day the downlink drains
only enough for about 50 %. The design is asking for roughly half again more imaging than
either the array or the ground segment can support.

**The binding constraint moves.** At one pass per orbit the downlink is not a constraint at
all and energy binds alone. At four passes a day the downlink binds first. Which number is
the answer therefore depends on a figure the model does not carry - how many usable passes
a day the ground segment provides - and that is a finding, not an inconvenience.

**The orbit-average draw agrees with the source.** 23.31 W computed here is the figure the
TrueSightSAT report reaches by a different route in section 5.2, where it records a per-orbit
deficit and recommends a reduced imaging duty cycle. The model now reproduces the report's
own conclusion from its own facts, which is the first time the two have been checked
against each other.

## What this analysis cannot do, and why

| Missing | Kind of gap | What it would take |
|---|---|---|
| Orbit period, altitude | Vocabulary | An `Orbit` concept with altitude and period; it is prose today |
| Passes per day | Vocabulary | A cadence property on `ContactOpportunity`, or a `GroundStation` that owns a pass schedule |
| Battery capacity, depth of discharge | Vocabulary | An `EnergyStore` concept; without it this is an average-power check, not an eclipse-survival check |
| Eclipse fraction | Vocabulary | Derivable from an orbit, once an orbit exists |
| Whether safe mode absorbs the spare time | Process | An operational policy decision, not a model fact |

Until the first four exist, this page answers Q2 **conditionally**: it gives the duty cycle
that closes an orbit-average balance under stated parameters, not the duty cycle that
survives an eclipse with a real battery. The honest form of the answer is the one with its
assumptions printed beside it.
