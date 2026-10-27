# Valdi320 — cleaning-jobs: original OOMWOO sweep reuse / next integration

**Source:** https://github.com/Valdi320/oomwoo-cleaning-jobs
**Verified revision:** https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/ca5f9ea1dd8d412bf2645f9f2bcb86addae8ff07
**Simulation evidence:** https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/UPSTREAM_SWEEP_SINGLE_NAV2_OWNER.md

## Current follow-up increment

- Reuses the **unmodified, existing** OOMWOO boustrophedon sweep planner's
  waypoint *geometry* without spinning its Nav2 action executor. This avoids
  two independent components commanding the same robot.
- Official test_room upstream plan: 409 source waypoints, adapted conservatively
  under the existing 5-cell clearance to 391 unique endpoints; 5 original
  positions rejected instead of silently weakening collision clearance.
- Real OOMWOO Gazebo + real Nav2 NavigateToPose, with **one action owner**:
  5 goals succeeded, **337 independently measured new coverage cells**
  recorded from the official, unmodified simulation ground-truth meter.
  A partial test only; not a complete room/home sweep.
- Bounded measured coverage disk selector skips a waypoint only when the
  *external meter* shows actually marked cells, never on Nav2 success alone.
- Prior work merged in PR #78 remains available: durable jobs and safe
  recovery, exact UUID journal, Nav2 KeepoutFilter sandbox, offline room/zone
  plans and small measured 9/9-cell spot regressions.
- 255/255 pure-Python regressions pass on the contributor's PC, Python -O
  and Python 3.12 in a no-network container.

## Still under development

Full-room autonomous execution, lost-coverage gap repair, multi-room
selection, battery/bin/mop dock-cycle, the final ROS2 action/service API,
and multi-map endurance tests are **not validated** by this incremental
pointer. Later implementation is maintained in the independent repo.

No robot hardware, MCU safety, physical dirt removal, or real charging
hardware has been tested. Battery input for OOMWOO's test_room integration
is synthetic. The maintainer should select the final public job API and
single owner of Nav2 actions before this work is deployed.

Apache-2.0; ChatGPT-assisted. This upstream PR changes only this
contributor-owned pointer and does NOT alter shared OOMWOO runtime/contracts.
