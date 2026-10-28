# Valdi320 — cleaning-jobs: current reviewer pointer

**Independent source (Apache-2.0):** https://github.com/Valdi320/oomwoo-cleaning-jobs

**Latest tested source SHA:** https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/694aaf3bb401a90973d4624fb711d7cb78b4973a

## Maintainer quick review (2026-10-28): reject replaced Nav2 journal writer lock inode

- **Safety defect reproduced:** opening and flock-locking a private `goal.json.lock` did not establish that its *pathname still named the locked inode*. A same-user lock rotation between open and acquisition could let another process lock a replacement while the original holder still issued a different Nav2 UUID. Of three deterministic new controls **two FAILED before the fix**, while stable-lock control passed.
- **Minimal fix:** after nonblocking exclusive flock, compare named lock `st_dev/st_ino` with the original descriptor and fail closed on rename, substitution or disappearance, **before** modifying the goal journal. Existing durable UUID-before-send ordering, Linux advisory lock and JSON schema unchanged.
- **Exact code:** https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/694aaf3bb401a90973d4624fb711d7cb78b4973a
- **Isolated comparison:** https://github.com/Valdi320/oomwoo-cleaning-jobs/compare/f8e0e6413a8105f877f971c085b65daafa185ba9...694aaf3bb401a90973d4624fb711d7cb78b4973a
- **Contributor branch:** https://github.com/Valdi320/oomwoo-cleaning-jobs/tree/contrib/cleaning-jobs-goal-lock-identity-20261028
- **Verification (Linux, executed):** new focal **3/3 PASS**, prior lock **4/4 PASS**, journal **15/15 PASS**, full regression **633/633 PASS**, `python3 -O` **633/633 PASS**, `git diff --check` PASS; **2 original counterexamples reproduced as failing pre-fix controls**.
- **CI:** [exact-SHA CI](https://github.com/Valdi320/oomwoo-cleaning-jobs/actions/runs/40782540933) status: completed; conclusion: success; jobs: unit (3.12):success, unit (3.10):success
- **Minimum reproduction:** checkout exact source SHA; `PYTHONPATH=src python3 -m unittest discover -s tests -p 'test_goal_journal_lock_identity_20261028.py' -q`; then `PYTHONPATH=src python3 -m unittest discover -s tests -q` and `PYTHONPATH=src python3 -O -m unittest discover -s tests -q`.
- **Machine evidence:** https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/694aaf3bb401a90973d4624fb711d7cb78b4973a/docs/evidence/goal_journal_lock_identity_20261028.json ; [review and exact limits](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/694aaf3bb401a90973d4624fb711d7cb78b4973a/docs/GOAL_JOURNAL_LOCK_IDENTITY_20261028.md).
- **Risk/compatibility:** no API or journal schema changes; relies on cooperating Linux processes and private local filesystem. A malicious actor replacing the name **after** the check, distinct journal paths, network FS and full global Nav2 arbitration remain unproven/unprotected.
- **Boundary:** SYNTHETIC Linux filesystem test **only**, not ROS2/Nav2/Gazebo, SBC/CPU power, real motors, MCU braking, charger or physical cleaning.
- **Integrate / revert:** review the isolated diff; import selectively into a canonical maintainer-owned cleaning controller after agreeing exclusive Nav2 owner and single journal path, then rerun Linux tests and ROS2 end-to-end. Revert an imported patch with a normal inverse commit; never force-merge or discard a live/orphaned Nav2 UUID.
- **Review / TODO:** [ ] decide canonical Nav2 single owner and durable journal location; [ ] run real ROS2 restart/rename recovery; [ ] measure full-stack low-cost ARM hardware; [ ] MCU-owned physical stop remains mandatory. No upstream runtime edits, no new PR, no new routine comment.

---

## Previous review snapshot (2026-10-28): durable sweep campaign cross-process serialization

- **Problem (reproduced):** two SweepCampaign writers can both read revision 1 and one erase a durable cancellation with a stale RUNNING write; a symlink/hardlink checkpoint is also accepted; FIFO substitution can hang startup. Three safe pre-fix negative controls observed 3 expected FAIL; the FIFO was not executed against vulnerable code.
- **Fix:** contributor-only stable private Linux flock sidecar around revision read/compare/fsync/atomic rename; startup recovery checks the exact revision and new creation cannot overwrite an unexpectedly existing checkpoint. Bounded (16 MiB), nonblocking no-follow regular-single-link file reads reject swapped links/special inodes.
- **Exact source:** https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/f8e0e6413a8105f877f971c085b65daafa185ba9
- **Focused compare:** https://github.com/Valdi320/oomwoo-cleaning-jobs/compare/b778d81786ab6d71684a455f00486e9fbe23d3af...f8e0e6413a8105f877f971c085b65daafa185ba9
- **Source branch:** https://github.com/Valdi320/oomwoo-cleaning-jobs/tree/contrib/cleaning-jobs-sweep-campaign-serialization-20261028
- **Tests on Linux Python:** new focal **7/7 PASS**, previous SweepCampaign **18/18 PASS**; full suite **630/630 PASS** and Python -O **630/630 PASS**; git diff --check PASS. [Exact-SHA GitHub CI](https://github.com/Valdi320/oomwoo-cleaning-jobs/actions/runs/40775706627) **PASS** (Python 3.10 and 3.12).
- **Reproduce:** checkout new source SHA; PYTHONPATH=src python3 -m unittest discover -s tests -p 'test_sweep_campaign_serialization_20261028.py' -q; then PYTHONPATH=src python3 -m unittest discover -s tests -q and PYTHONPATH=src python3 -O -m unittest discover -s tests -q.
- **Evidence:** [machine-readable 7+630 results and pre-fix controls](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/f8e0e6413a8105f877f971c085b65daafa185ba9/docs/evidence/sweep_campaign_serialization_20261028.json); [review/spec/compatibility](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/f8e0e6413a8105f877f971c085b65daafa185ba9/docs/SWEEP_CAMPAIGN_SERIALIZATION_20261028.md).
- **Compatibility and risk:** schema-1 JSON is unchanged. File and lock must be private single-link regular files; legacy permissive checkpoint files will fail closed until owner inspects/repairs permissions. Busy lock rejects transitions instead of silently rearming. Local Linux advisory flock does NOT provide global exclusive Nav2 goal ownership across distinct processes/paths or protect against a malicious noncooperating writer.
- **Integrate / revert:** examine isolated source/test diff, import only into the canonical owned cleaning-job controller after agreeing checkpoint ownership, rerun the suite and real ROS2/Gazebo recovery; revert imported commit normally if required. Do not delete unresolved Nav2 journals. This contribution does not touch main, upstream runtime, firmware, CAD, motor or charging control.
- **Boundary:** SYNTHETIC local Linux filesystem tests only for this patch; NOT ROS2/Nav2/Gazebo, ARM SBC, physical hardware, energy or real cleaning.
- **Review decisions/TODO:** [ ] canonical cleaning-job API; [ ] unique Nav2 action owner; [ ] real ROS2/Gazebo concurrency/crash test; [ ] eventual MCU-owned physical interlocks.

---

## Previous review snapshot (2026-10-28): map-frame provenance safety

- **Problem reproduced before patch:** 3/5 newly added tests FAILED: when an OccupancyGrid kept identical pixels but changed world origin (x/y) or yaw, the same adapted cell-index sweep plan was accepted, including a durable campaign that wrote a session. A cell index alone does **not** identify the same physical floor point after map-origin changes.
- **Fix:** SHA-256 geometry binding (frame, grid size, exact resolution, origin x/y and yaw) added to `SafeSweepPlan` and its plan digest; reject stale plans in the measured selector and durable campaign **before** choosing goals or writing a checkpoint.
- **Source commit:** https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/b778d81786ab6d71684a455f00486e9fbe23d3af
- **Focused diff:** https://github.com/Valdi320/oomwoo-cleaning-jobs/compare/8123edd8b30e01fcbb46470af4d3470aa4326473...b778d81786ab6d71684a455f00486e9fbe23d3af
- **Independent source branch:** https://github.com/Valdi320/oomwoo-cleaning-jobs/tree/contrib/cleaning-jobs-plan-frame-binding-20261028
- **PASS (Linux x86-64, Python 3.14.3):** 6/6 focused post-fix tests; 623/623 full regressions normal and 623/623 under `python3 -O`; `git diff --check` PASS. Pre-fix negative controls: 3 expected FAIL, 2 PASS (not regressions on fixed SHA).
- **Exact source CI:** [GitHub Actions 40766758124](https://github.com/Valdi320/oomwoo-cleaning-jobs/actions/runs/40766758124), Python 3.10 and 3.12 **2/2 PASS**.
- **Reproduce:** checkout source commit above; `PYTHONPATH=src python3 -m unittest discover -s tests -p 'test_sweep_plan_geometry_provenance_20261028.py' -v`; then `PYTHONPATH=src python3 -m unittest discover -s tests -q` and `PYTHONPATH=src python3 -O -m unittest discover -s tests -q`.
- **Machine evidence:** [JSON](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/b778d81786ab6d71684a455f00486e9fbe23d3af/docs/evidence/sweep_plan_frame_binding_20261028.json); [review/replay constraints](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/b778d81786ab6d71684a455f00486e9fbe23d3af/docs/SWEEP_PLAN_FRAME_PROVENANCE_20261028.md).
- **Compatibility/risk:** plan-digest format intentionally changes; old checkpoints and direct `SafeSweepPlan` construction cannot silently migrate. Re-adapt from current validated map and reconcile outstanding Nav2 actions externally before new jobs. Exact float checks may reject equivalent transforms after lossy round-trip; that is conservative.
- **Integration/revert:** evaluate isolated source patch, apply/cherry-pick into an explicitly owned module only after canonical ROS2 interface/owner review, rerun focal + full tests; revert imported commit normally if necessary. No shared upstream runtime, firmware, CAD, charging logic or motor endpoints were modified.
- **Evidence boundary:** **synthetic/headless pure Python only** for this patch; NOT a ROS2/Gazebo, ARM SBC, physical hardware, cleaning-quality, MCU or battery validation.
- **Reviewer checklist / TODO:** [ ] confirm chosen cleaning job API; [ ] establish a single Nav2 action owner; [ ] verify map geometry provenance at real ROS2 lifecycle/restarts; [ ] verify Nav2/Gazebo integration and eventual safe physical hardware under MCU interlocks.

The previous measured CPU resource optimization remains documented below; no performance improvement is claimed for this safety patch.

---

## Previous review snapshot (2026-10-28): measured resource fast path

- **Problem:** a pure-Python measured-sweep selector recalculated all traversable cells repeatedly during disk inspection. This creates avoidable full-map work, especially in a long multi-room job.
- **Fix:** calculate an immutable local traversability snapshot once per select call, reuse the same snapshot for scope, exclusions and every sampled sweep cell; preserve all coverage evidence and keepout checks.
- **Diff:** https://github.com/Valdi320/oomwoo-cleaning-jobs/compare/ac4ccd7c83691cab589ad21653886a906b6473eb...8123edd8b30e01fcbb46470af4d3470aa4326473
- **Owner source branch:** https://github.com/Valdi320/oomwoo-cleaning-jobs/tree/contrib/cleaning-jobs-selector-resource-baseline-20261028
- **Synthetic PC benchmark:** identical 30×18, three virtual rooms, 530/530 credited floor cells, 125 synthetic goals, no forbidden cells entered; two trials per revision, one x86 desktop core pinned: median wall time **9.104s before / 0.796s after (91.3% less)**. Process peak RSS **23168 KiB before / 23832 KiB after**; **NO memory saving claimed**.
- **Validation:** 3/3 additional focused tests PASS; full local Python test suite **617/617 PASS**, Python -O **617/617 PASS**; [exact-SHA CI Python 3.10 and 3.12, both PASS](https://github.com/Valdi320/oomwoo-cleaning-jobs/actions/runs/40757977174).
- **Machine evidence:** https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/8123edd8b30e01fcbb46470af4d3470aa4326473/docs/evidence/selector_one_snapshot_resource_20261028.json
- **Reproduce:** checkout the source SHA; run `PYTHONPATH=src python3 tests/bench_home_resources.py --seed 0 --one-core` and `PYTHONPATH=src python3 -m unittest discover -s tests -q`, then repeat with `python3 -O`.
- **Limits:** host-specific two-trial synthetic Python measurement, **not full ROS2, Nav2, Gazebo, low-cost ARM SBC or physical robot**. No validated power, thermal, minimum RAM, battery, motors, MCU watchdog or cleaning quality; cheap hardware selection must wait for a full-stack representative workload. Runtime behavior is not integrated into official OOMWOO.
- **Integration / rollback:** reviewer may import the isolated contributor source change after inspecting the small diff and verifying tests; normal `git revert` can back it out. No forced merge, upstream Nav2 runtime or third-party files changed.
- **Outstanding maintainer decisions:** canonical cleaning-job API and exclusive Nav2 action owner; measurement of full-stack CPU, RAM, latency and power on affordable hardware.

Historical milestones below are snapshots, not the current benchmark or CI result.

---

## Major verified deliverables

- The UNMODIFIED OOMWOO boustrophedon waypoint generator is reused
  without starting its own navigation executor: 409 original targets,
  391 conservatively admitted distinct waypoints, five original
  unsafe/unresolved targets explicitly flagged (not falsely completed).
- A bounded REAL OOMWOO Gazebo + Nav2 action trial issued exactly six
  waypoints, all succeeded, with 348 NEW cells from the ORIGINAL
  simulated ground-truth coverage meter; 246 cells were durably
  retained across a SYNTHETIC battery/bin/mop station service pause.
  This does NOT prove an entire room was cleaned.
- Persistent multi-step sweep campaigns, selected-room/whole-home
  orchestration, gap analysis, fixed-size retry bounds, checkpoint
  restart handling and write-ahead coverage journals were developed
  and tested with deliberate interrupted-write fault injection.
- 12/12 separate deterministic SYNTHETIC three-room home scenarios
  completed all eligible cells with no forbidden-zone entries, while
  exercising simulated dock cycles and a controller restart.
  These are NOT 12 Nav2/Gazebo full-home simulations.
- An experimental typed ROSIDL StartCleaning / ControlCleaning /
  GetCleaningStatus package builds offline and passes actual ROS2
  middleware calls against a FAKE Nav2 action server, including
  duplicate-request and cancel/result gating.
- REAL Nav2 Jazzy/Gazebo KeepoutFilter originally passed two static
  prohibited-zone robot movements. This follow-up additionally
  demonstrates LIVE static-zone replacement using LoadMap:
  old 220-cell zone replaced with new 198-cell zone, confirmed by
  two distinct global/local post-filter updates and time-fenced
  TF2 map<-odom conversion; NO motor/navigation command during edit.
- Subsequent actual ComputePathToPose accepted a 32-pose route to
  the former forbidden zone but stopped 0.40 m short of the new
  forbidden destination on a truncated 15-pose path. The application
  must reject forbidden goals before dispatch, regardless of Nav2's
  success code.
- 383/383 pure-Python regressions PASS, including Python -O and
  Python 3.12, plus separate real ROS2 protocol and Nav2/Gazebo tests.

## Reproducible reports

- [Ten milestones, reality labels and limits](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/TEN_SUBSTANTIAL_MILESTONES_20261027.md)
- [Verified live KeepoutFilter edit](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/REAL_NAV2_DYNAMIC_KEEPOUT_20261027.md)
- [Six real Nav2 goals / synthetic station](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/evidence/upstream_sweep_6goals_hardened_20261027.json)
- [12 synthetic homes](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/evidence/synthetic_multihome_12cases_20261027.json)

## Critical remaining work / maintainer decisions

These are independent test modules; a production full-room or
multi-room queue is NOT integrated into the persistent OOMWOO ROS2
node. The typed services are an isolated PRIVATE prototype, not a
canonical approved OOMWOO interface. The verified KeepoutFilter
overlay is NOT in OOMWOO's stock bringup. Real dock/recharge hardware,
MCU safety, physical dirt removal, actual sensor-gated cleaning and
multi-room long-duration Nav2/Gazebo validation remain outstanding.

Please advise (1) the canonical job API and (2) the exclusive Nav2
action owner so this can be integrated without two autonomous planners.

ChatGPT-assisted, reproducibly tested on Linux in an isolated offline
OOMWOO Jazzy/Gazebo environment. Upstream changes ONLY this owned
contribution pointer; shared runtime and other contributors stay unchanged.


### Latest follow-up: measured virtual-room completion in REAL Gazebo

Independent source commit: https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/1cd6b5eb5753a832113e6a3c416e6fc084ecde80

The opt-in persistent ROS2 controller now schedules the **original OOMWOO
sweep-plan geometry** under one Nav2 owner, saves the per-room plan and
verifies coverage from the unmodified official Gazebo ground-truth meter.
For one **operator-defined digital region** in an OPEN test_room (NOT a
physically separated apartment room), actual checkpoints show
**171/171 eligible floor cells externally measured**, BOTH durable
CleaningJobs and SweepCampaign states COMPLETED, 11 Nav2 requests in
the log and the final UUID terminal CANCELED.

Added guarded gap-route proposals (Nav2 endpoint tolerance can leave
a narrow dirty fringe) and a software-only dock geometry / exclusive
cmd_vel owner handoff contract. 425/425 pure Python regressions PASS.
This is NOT proof that the real robot can safely hand motors between
Nav2 and dock-drive, charge, empty its dustbin, wash mop or remove dirt.

Report: https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/REAL_ROOM_AND_DOCK_BOUNDARY_20261027.md

Physical station service, real motor mux/Nav2 dock arbitration, MCU
safety and whole multi-room Gazebo endurance remain for integration.

## Follow-up 2026-10-27: TWO COMPLETE digital zones in REAL OOMWOO Gazebo

**Published implementation and tests:**
https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/471a483f51fed1ba727c5befa49b39e54ac93fc3

The same persistent simulation-only cleaning controller completed two
different operator-defined, disjoint virtual regions on the official
OOMWOO test_room map in ONE selected-room job. These regions are on the
SAME OPEN FLOOR, not separated by architectural walls.

- Area A: **80/80** externally measured cells.
- Area B: **40/40** externally measured cells.
- Total: **120/120** independently measured eligible cells, 100%
  of exactly the selected virtual map scope.
- REAL OOMWOO Nav2 Jazzy: **18 accepted goals**; original OOMWOO
  ground-truth meter: **200 observed frames**; **12** bounded
  gap-repair routes proposed (no fabricated cleaning credit).
- Both durable CleaningJobs and SweepCampaign checkpoints reached
  COMPLETED with identical 120-cell credited sets. CoverageWAL had
  20 applied events and zero pending. The final UUID was terminal
  CANCELED after external completion; no active goal remained.
- Earlier 117/120 partial/timeout runs are preserved separately and
  were not misreported as successful.

An experimental isolated ROS2 dock velocity relay now uses the fsync'd
CURRENT Nav2 goal UUID, observed ACTIVE during this lease and then
matching terminal before transferring from Nav2 to dock_drive. Old
terminal statuses cannot authorize a new run. Foreign Nav2 action,
battery-low, empty/stale status and expired owner commands force STOP.
Actual ROS2 middleware/typed control test PASS, with SYNTHETIC
status/odometry/battery and NO MOTOR CONNECTIONS. The separate
UNMODIFIED upstream OOMWOO 2D docking harness also passed 7/7
simulated geometric approaches, not physical charging.

**437/437** headless regressions PASS on host, optimized Python and
Python 3.12; existing ROS2 probe matrix remains green.

- [Actual two-zone Gazebo evidence](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/evidence/real_two_virtual_zones_complete_20261027.json)
- [ROS2 exact-UUID dock relay evidence](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/evidence/ros2_exact_uuid_dock_relay_20261027.json)
- [Integration and production limits](https://github.com/Valdi320/oomwoo-cleaning-jobs/blob/main/docs/REAL_TWO_ZONES_AND_DOCK_RELAY_20261027.md)

**Production remainers:** physically divided furnished-room navigation,
real exclusive robot cmd_vel remaps and MCU safety; charge current,
auto-empty and mop-wash provider protocols plus hardware sensors; actual
vacuum/mop actuator-state coverage, full-home endurance, and canonical
OOMWOO ROS2 API/maintainer approval. Lab relay publishes to a SAFE,
NON-MOTOR topic only. This is not 100% of a hardware cleaning robot.


### Additional nonhardware physical coverage-admission contract

Source: https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/0bf06d549374df5c55eb6c76a453f8b28adf9b6c

PhysicalCoverageAdmission is a fail-closed, pure-Python policy that
rejects credit for floor cells unless fresh and map-matching
localization, an accepted active navigation goal, independently
verified vacuum current and/or mop actuator contact, and a trusted
coverage source all agree. It has 22 additional negative/positive
tests; the entire regression suite passes 459/459 tests in normal,
optimized and Python 3.12-container modes.

**No real vacuum-current sensor, mop-switch, source authentication or
physical actuator has been connected**. This does NOT certify that
the hardware actually cleans. Production must supply independent
signed/trusted sensor metadata and MCU-owned safety interlocks.
