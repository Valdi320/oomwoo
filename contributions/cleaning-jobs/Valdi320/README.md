# Valdi320 — cleaning-jobs, ten substantial engineering milestones

Independent Apache-2.0 implementation:
https://github.com/Valdi320/oomwoo-cleaning-jobs

**Verified source commit:** https://github.com/Valdi320/oomwoo-cleaning-jobs/commit/c9fab39542c32377566850529c9350ac7cc5b4b9

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
