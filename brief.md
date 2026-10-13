# Brief: a bell-span-load flying wing

## Purpose

A finless flying wing in the spirit of NASA's Prandtl-D: a bell-shaped span load for proverse yaw without a fin,
nice handling, and as good a glide as the rest allows. 3D printed in LW-PLA, pusher prop on a BLDC motor (not
modelled; only its mass is placed). Conception stage: no parts chosen yet.

## Priorities

The three that matter most, last to give: **proverse yaw** from the elevons, **root stalls before the tips**, and a
**minimum speed of 6.5 m/s or lower**. If one of them must give, ask the human; they have not ordered them.

**Handling** comes next: turns entered without a large adverse skid, especially slow (landing pattern), measured by
`rollin_sideslip`. It ranks above glide.

Before any requirement gives, every free choice is used: planform, sweep, taper, aspect ratio, twist, airfoils,
thickness, a yehudi or bat tail (per-section chord and offset), battery mass and position. Then, in order: the
Reynolds floor from 60k down to 55k; then the wants. Span beyond 1.8 m or mass beyond 1.8 kg: ask the human.

Wants, reported and negotiable: glide ratio at least 15 (above it as high as possible; glide above 15 may be spent
on proverse yaw, turn entry and stall robustness, below 15 ask the human); elevons near neutral at 9 m/s; lighter
is better; the smallest span that meets the musts (1.8 m is a limit, not a target; spans below about 1.7 m looked
unpromising to the human).

## Design rules (not checkable; designers hold to them)

- Flying wing only: no fin, no winglets, no inlets, no vertical surface of any kind.
- Dihedral 0 everywhere (a straight spar).
- Exactly two control surfaces: elevons on the outer wing, mixed for pitch and roll. In `plane.toml` they are two
  `[[surface]]` entries on the same flaps: `pitch` (symmetric) and `roll` (antisymmetric). Throw at most ±20°.

## Assumptions

- Structure: printed LW-PLA, 87 kg/m3 of the wing's enclosed volume (`structure_density`); the human's
  pessimistic figure, mass over the volume inside the skin.
- Battery: 0 to 0.4 kg (no floor; flight time is not a requirement). Its position is free, including forward of the
  root leading edge (on a nose or boom).
- Motor and prop 0.07 kg at the root trailing edge (pusher); ESC 0.03 kg; receiver 0.01 kg; two 9 g servos at the
  elevons; carbon spar 0.03 kg. Typical estimates; no parts chosen yet.
- Sea level, standard atmosphere (flow5's air: density 1.225, nu 1.5e-5). No fuselage or propulsion in the model.

## Interpretations

- "Statically stable": static margin of at least 3% MAC, a small buffer above neutral.
- "Not uncontrollable; damping nice but not required":
  - short period must not grow (`sp_tdouble` ≥ inf): its cycle is a fraction of a second, too fast for a pilot,
    and a statically stable flying wing damps it heavily anyway;
  - Dutch roll may grow, doubling in no less than 4 s (`dr_tdouble`): a cycle of a couple of seconds that a pilot
    calms with the elevons; requiring damping could be hard without a fin;
  - phugoid may grow, doubling in no less than 8 s (`ph_tdouble`): a cycle of about 4 s at these speeds, so it
    doubles over two swings; more lenient because the solver reads gliders' phugoid damping low;
  - spiral may grow, doubling in no less than 4 s (`spiral_tdouble`).
- "Re floor 60,000, or at most 55,000": `re_min` over the trim line down to the minimum speed, every strip.
  55,000 is a must, 60,000 a want, the first thing to give.
- "Root stalls before the tips" must also survive a ±0.05 error in the sections' Cl max, the adversarial pair
  shift included (root +0.05, tip −0.05), on the wake aligned with the free stream: XFoil's known Cl-max error is
  0.02 to 0.07 on comparable sections. The brief block cannot express it yet (uav#113); designers verify it with
  `f5 envelope --wake aligned --limit-shift 0.05` and report it.
- "Proverse yaw": the static `proverse_yaw` row (the elevons' yaw is into the roll).
- "Nice handling, clean turn entry": `rollin_sideslip`, a roll from the trim to 30° of bank on an elevon step,
  continued 3 s after the bank with the elevons released, peak sideslip, airfoil drag included. Basis of the
  thresholds: both of the tools' models (flow5 and the lifting-line second model) read NASA's Prandtl-D — a
  finless bell-spanload wing that flew acceptably — with adverse yaw due to roll rate, near-zero or negative
  weathercock stiffness and, in flow5, a 38° peak on its slowest judgeable roll (about 9 s to 30°). The models are
  therefore pessimistic for this class and absolute values are not trusted; the anchor is relative.
  - must: on a gentle roll of about that duration (a 1.6° step, about 9 s to 30° on this design), peak sideslip
    no worse than the Prandtl-D's 38°;
  - want: on a brisk 10° step (about 1.5 s to 30°), peak sideslip ≤ 20°.
- "Landing / minimum speed 6.5 m/s": `speed_min`, the trim-line speed at the first section's stall onset. It is
  conservative; the plane flies somewhat slower before it actually stalls.
- "Roughly cruises without trim": elevons within ±1° at 9 m/s, a want. 9 m/s is a reference speed, not a cruise
  target.

```toml brief
[plane]
mass_max = 1.8
span = [1.5, 1.8]

[conditions]
trim_surface = "pitch"
roll_surface = "roll"
cruise_speed = 9.0

[[conditions.case]]
name = "trim"
path = "trim"

[[require]]
metric = "proverse_yaw"
equals = true
why = "the point of the bell span load: no fin"

[[require]]
metric = "stall_order"
equals = ["root", "tip"]
at = "trim"
why = "root stalls before the tips"

[[require]]
metric = "speed_min"
max = 6.5
at = "trim"
why = "landing speed 6.5 m/s or lower"

[[require]]
metric = "static_margin"
min = 0.03
why = "statically stable, with a small buffer"

[[require]]
metric = "re_min"
min = 55000
at = "trim"
why = "the 2D polars are not trusted below Re 55k"

[[require]]
metric = "re_min"
min = 60000
at = "trim"
priority = "want"
why = "the human's preferred floor; gives first"

[[require]]
metric = "sp_tdouble"
min = inf
why = "must not grow: too fast for a pilot"

[[require]]
metric = "dr_tdouble"
min = 4
why = "may grow, slowly enough for a pilot to calm it"

[[require]]
metric = "ph_tdouble"
min = 8
why = "may grow over two swings; a pilot corrects pitch anyway"

[[require]]
metric = "spiral_tdouble"
min = 4
why = "a divergence slow enough for a pilot to correct"

[[require]]
metric = "rollin_sideslip"
bank = 30
input = { type = "step", deflection = 1.6 }
max = 38
why = "gentle turn entry no worse than NASA's Prandtl-D in the same model (38 deg at a ~9 s roll to 30 deg)"

[[require]]
metric = "rollin_sideslip"
bank = 30
input = { type = "step", deflection = 10 }
max = 20
priority = "want"
why = "brisk turn entry: a clean entry on a firm input"

[[require]]
metric = "span"
max = 1.5
priority = "want"
why = "smallest span that works; the 1.8 m ceiling is a limit, not a target"

[[require]]
metric = "trim_deflection"
min = -1.0
max = 1.0
priority = "want"
why = "roughly cruises without trim at 9 m/s"

[[require]]
metric = "glide_ratio"
min = 15.0
priority = "want"
why = "floor 15; above it as high as possible, a tie-breaker that may be spent on yaw, roll-in and stall robustness"
```
