# Robotic Arm Sorting System · Vision-Guided 6-DOF Picker

> **Project status: completed during 2024 summer internship (Jun–Aug 2024).**  
> Production code remains in the company intranet (NDA covered); this repository documents the methodology, system design, and empirical results that the public-facing version of this work was based on.

A vision-guided 6-DOF robotic-arm sorting workstation for industrial bin-picking.
YOLOv5 detects and localizes workpieces on a conveyor; eye-in-hand calibration
projects pixel coordinates into the arm's base frame; MoveIt plans collision-free
grasps; a 6-axis arm executes picks into two material bins. Continuous runs
demonstrated **98% pick success and 7-second cycle time** under production
conditions.

---

## 1. System Overview

```
                        Workpiece
                            │
                            ▼
                  ┌──────────────────┐
                  │  RGB-D Camera    │   Intel RealSense D435
                  │  (eye-in-hand)   │   ──>  RGB + Depth stream
                  └────────┬─────────┘
                           ▼
                  ┌──────────────────┐
                  │   YOLOv5s Det.   │   Mosaic + MixUp augmentation
                  │   mAP@0.5 = 96.2 │   multi-scale training
                  └────────┬─────────┘
                           │  (u, v, Z)
                           ▼
                  ┌──────────────────┐
                  │  Hand-Eye Calib. │   pixel → base-frame transform
                  │  eye-in-hand     │   (Tsai–Lenz, 9-pose calibration)
                  └────────┬─────────┘
                           │  (x, y, z, roll, pitch, yaw)
                           ▼
                  ┌──────────────────┐
                  │  MoveIt Planner  │   OMPL RRTConnect
                  │  + ROS Control   │   collision-free trajectory
                  └────────┬─────────┘
                           │  joint trajectory
                           ▼
                  ┌──────────────────┐
                  │  6-DOF Arm       │   serial-port control @ 50 Hz
                  │  + Gripper       │   pneumatic gripper
                  └──────────────────┘
                           │
                           ▼
                     Bin A  /  Bin B
```

## 2. Tech Stack

| Layer | Tools |
|---|---|
| Vision | YOLOv5 (PyTorch), Mosaic + MixUp augmentation, multi-scale training |
| Calibration | eye-in-hand (Tsai–Lenz), depth-camera → base-frame transformation |
| Planning | MoveIt + OMPL (RRTConnect), collision scene from depth cloud |
| Control | ROS Noetic, `ros_control` joint trajectory controller, 50 Hz loop |
| Hardware | 6-DOF arm (vendor-anonymized), Intel RealSense D435, pneumatic gripper |
| Ops | Multi-target priority queue, automatic retry on grasp failure |

## 3. Key Results

| Metric | Value | Note |
|---|---|---|
| Pick success rate (online) | **98 %** | 200 picks over 8 h continuous run |
| Cycle time (single pick) | **7.0 s** | detect + plan + execute + return |
| YOLOv5 mAP@0.5 (offline) | 96.2 % | 500-image test set |
| Continuous uptime | 8 h+ no fault | production shift |

The 2 % failure rate is dominated by **multi-target occlusion** and **specular
workpieces** (reflective surface degrading depth at the gripper). Mitigation
in v1.1: switch to a coarse-to-fine pose estimator when inter-object distance
< 50 px, which lifted system-level success by ~5 pp.

## 4. Engineering Highlights

1. **Hand-eye calibration with depth fusion.** Pixel `(u, v)` from YOLOv5 is
   combined with depth `Z` from the RGB-D camera, then projected to the arm's
   base frame via a rigid transform solved by the Tsai–Lenz method (9-pose
   calibration, reprojection error < 0.6 px).

2. **Failure-tolerant scheduling.** A priority queue ranks detected targets by
   confidence × isolation margin. On grasp failure, the system does *not*
   retry the same target — it pops the next easiest target and resumes. This
   converts "give up" into "skip and continue", lifting the system success
   rate without inflating per-target latency.

3. **Mosaic + MixUp for small / occluded objects.** Many workpieces arrive
   partially occluded by adjacent parts. Mosaic and MixUp are applied at
   training time to force the network to learn from incomplete views,
   improving recall on the hardest decile by ~8 pp.

4. **MoveIt + ROS separation.** Detection runs at 10 Hz, planning at 2 Hz on
   demand, control at 50 Hz. The three loops are decoupled through ROS
   topics, so a slow detection does not stall the arm in the middle of a
   trajectory.

## 5. Repository Contents

This repository is the **public, code-free** version of the system. It
contains the design documents only:

```
robotic-arm-sorting-system/
├── README.md              ← this file
├── ARCHITECTURE.md        ← detailed system design + sequence diagrams
├── PERFORMANCE.md         ← test methodology, data tables, error analysis
├── METHODOLOGY.md         ← calibration procedure, training recipe
└── assets/                ← placeholders for figures / diagrams
```

The production implementation (YOLOv5 inference, MoveIt configs, ROS launch
files, calibration capture scripts) lives behind the company's internal Git
and cannot be redistributed under the internship NDA.

## 6. Why the code is not here

The work was done as part of a paid summer internship (2024-06 → 2024-08).
The deliverable code is hosted on the company's private infrastructure and
is bound by the standard intern NDA covering proprietary hardware drivers,
calibration data and customer-identifying images. Sharing the public-facing
methodology, diagrams and results is permitted; sharing the production code
is not.

If you would like a deeper technical discussion during an interview (e.g.
the calibration math, the OMPL planner choice, or the failure-recovery
queue), I am happy to walk through it in person.

---

## 7. Contact

GitHub: [JingHao-Leon](https://github.com/JingHao-Leon)  
Email:  262\*\*\*\*56@qq.com  
Phone:  159\*\*\*\*0000

(Contact details redacted for spam protection — full info on request.)
