# Brief: a bell-span-load flying wing

## Purpose

A finless flying wing in the spirit of NASA's Prandtl-D: a bell-shaped span load for proverse yaw without a fin,
and as good a glide as the rest allows. 3D printed in LW-PLA, pusher prop on a BLDC motor (not modelled; only its
mass is placed). Conception stage: no parts chosen yet.

## Priorities

The three that matter most, last to give: **proverse yaw** from the elevons, **root stalls before the tips**, and a
**minimum speed of 6.5 m/s or lower**. If one of them must give, ask the human; they have not ordered them.

Before any of those gives, the designers use every free choice: planform, sweep, taper, aspect ratio, twist,
airfoils, thickness, a yehudi or bat tail (via per-section chord and offset), battery mass and position, mass in
general. Then, in order: the Reynolds floor from 60k down to 55k; then the wants. Span beyond 1.9 m or mass beyond
1.8 kg: ask the human.

Wants, reported but negotiable: elevons near neutral at 9 m/s; glide ratio (no target, as high as possible);
lighter is better.

## Design rules (not checkable; designers hold to them)

- Flying wing only: no fin, no winglets, no inlets, no vertical surface of any kind.
- Dihedral 0 everywhere (a straight spar).
- Exactly two control surfaces: elevons on the outer wing, mixed for pitch and roll. In `plane.toml` they are two
  `[[surface]]` entries on the same flaps: `pitch` (symmetric) and `roll` (antisymmetric). Throw at most ±20°.

## Assumptions

- Structure: printed LW-PLA, 87 kg/m3 of the wing's enclosed volume (`structure_density`); the human's
  pessimistic figure, confirmed as mass over the volume inside the skin.
- Battery: 0 to 0.4 kg, the designers' choice (no floor; flight time is not a requirement). Its position is free,
  including forward of the root leading edge (on a nose or boom).
- Motor and prop 0.07 kg at the root trailing edge (pusher); ESC 0.03 kg; receiver 0.01 kg; two 9 g servos at the
  elevons; carbon spar 0.03 kg. Typical estimates, accepted by the human; no parts chosen yet.
- Sea level, standard atmosphere. No fuselage or propulsion in the model.

## Interpretations

- "Statically stable": static margin of at least 3% MAC, a small buffer above neutral.
- "Not uncontrollable; damping nice but not required": spiral time to double at least 4 s if unstable. Short
  period: must not grow (`sp_tdouble`, a stable mode reads inf), the interviewer's recommendation, accepted: its
  cycle is a fraction of a second, too fast for a pilot, and a statically stable flying wing damps it heavily
  anyway. Dutch roll: may grow, time to double at least 4 s (`dr_tdouble`), as the spiral; its cycle is a couple
  of seconds and a pilot calms a slow growth with the elevons; requiring damping could be hard without a fin.
  Interviewer's recommendation, accepted. Phugoid: damping not required; time to double still to agree
  (`ph_tdouble`). The human's rule: no stand-in requirement; no design work until it is agreed.
- "Re floor 60,000, or at most 55,000": `re_min` over the trim line down to the minimum speed, every strip
  (flow5's air, nu 1.5e-5). 55,000 is a must, 60,000 a want: the first thing to give, per the priorities.
- "Landing / minimum speed 6.5 m/s": `speed_min`, the trim-line speed at the first section's stall onset. It is
  conservative; the plane flies somewhat slower before it actually stalls.
- "Roughly cruises without trim": elevons within ±1° at 9 m/s, a want. 9 m/s is a reference speed, not a cruise
  target.

```toml brief
[plane]
mass_max = 1.8
span = [1.5, 1.9]

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
metric = "spiral_tdouble"
min = 4
why = "a divergence slow enough for a pilot to correct"

[[require]]
metric = "trim_deflection"
min = -1.0
max = 1.0
priority = "want"
why = "roughly cruises without trim at 9 m/s"

[[require]]
metric = "glide_ratio"
min = 0.0
priority = "want"
why = "reported only: as high as possible"
```
