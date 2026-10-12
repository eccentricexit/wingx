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
general. Then, in order: the Reynolds floor from 60k down to 55k; then the wants. Span beyond 1.8 m or mass beyond
1.8 kg: ask the human.

Handling (the human, 2026-10-12): "we want nice behaviour/handling": turns entered without a large adverse
skid, especially slow (landing pattern). Measured by `rollin_sideslip` (comparative until recalibrated); it ranks
above glide, which may be spent on it down to the floor.

Wants, reported but negotiable: elevons near neutral at 9 m/s; glide ratio at least 15 (above it as high as possible; glide above 15 may be spent on proverse yaw, roll-in and stall robustness, below 15 ask the human);
lighter is better; the smallest span that meets the musts. The 1.8 m ceiling is a limit, not a target. First
attempt starts at 1.7 m (the human's call: lower spans look unpromising), then moves either way as the checks say.

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
  Interviewer's recommendation, accepted. Phugoid: may grow, time to double at least 8 s (`ph_tdouble`): its
  cycle is about 4 s at these speeds, so it doubles over two swings, easy for a pilot who corrects pitch anyway;
  more lenient than the Dutch roll because the solver reads gliders' phugoid damping low. Interviewer's
  recommendation, accepted.
- "Re floor 60,000, or at most 55,000": `re_min` over the trim line down to the minimum speed, every strip
  (flow5's air, nu 1.5e-5). 55,000 is a must, 60,000 a want: the first thing to give, per the priorities.
- "Root stalls before the tips" must survive a ±0.05 error in the sections' Cl max, the adversarial pair shift
  included (root +0.05, tip −0.05), on the wake aligned with the free stream: inside XFoil's known Cl-max error
  (0.02 to 0.07 on the E387). Interviewer's recommendation, accepted. The brief block cannot express it yet
  (uav#113); designers verify it with `f5 envelope --wake aligned --limit-shift 0.05` and report it.
- "Proverse yaw" in practice: a roll from the trim to 30° of bank on a 10° elevon step, peak sideslip at most 3°
  (`rollin_sideslip`, airfoil drag included; its note brackets flow5's Cnp underestimate). A want for now: the
  3° is a first guess; once Prion and the next variant give a baseline, the human sets the threshold and it
  becomes a must. The static `proverse_yaw` row stays as a must.
  Baseline (2026-10-12): Prion reads 27° (27–32° over the Cnp bracket) on this input, reaching 30° in 1.84 s, so
  3° is the wrong scale for a finless wing on a brisk held step, and the linear model is far outside its range at
  that size. The human delegated the recalibration to the interviewer (autonomous iteration): the row is used
  comparatively (lower is better) until a diagnostic shows which levers move it; then input and threshold are
  reset here with their basis.
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
input = { type = "step", deflection = 10 }
max = 3
priority = "want"
why = "rolls into a turn without the nose swinging out; want until a baseline sets the threshold"

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
