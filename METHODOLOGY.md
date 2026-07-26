# Methodology · Calibration, Training, Planning

## 1. Hand–Eye Calibration (eye-in-hand, Tsai–Lenz)

1. Mount a checkerboard (8×6, 25 mm squares) on the workspace.
2. Move the arm to **9 distinct poses** that all see the full checkerboard.
3. For each pose, record:
   - End-effector pose `T_gripper2base` (forward kinematics)
   - Checkerboard pose `T_target2camera` (PnP solve from the 2D detection)
4. Solve `AX = XB` for `X = T_camera2gripper` using dual quaternions.
5. Verify reprojection error < 0.6 px on a held-out pose.

## 2. YOLOv5 Training Recipe

| Hyperparameter | Value |
|---|---|
| Backbone | YOLOv5s (CSP-Darknet53) |
| Input | 640×640 |
| Optimizer | SGD, momentum 0.937, weight decay 0.0005 |
| LR schedule | cosine, 0.01 → 0.001 |
| Epochs | 300 |
| Batch size | 32 |
| Augmentation | Mosaic (p=1.0), MixUp (p=0.5), HSV jitter, random affine |
| Loss | YOLOv5 CIoU + objectness + class |

Dataset: 12 classes of small mechanical parts, ~5,000 labeled images,
80/10/10 train/val/test split. Annotation in Roboflow, exported as YOLO format.

## 3. MoveIt Planner

- **Library**: OMPL with **RRTConnectkConfigDefault**
- **Planning time**: 0.2 s budget, fallback to RRT* if no path
- **Collision checking**: FCL, with the depth point cloud voxelized at
  5 mm resolution as the obstacle field
- **Post-processing**: shortcutting + path smoothing, total smoothed path
  typically < 1.2× the geometric shortest path

## 4. Control Loop

- Joint trajectory streamed to `position_controllers/JointTrajectoryController`
- 50 Hz control rate, position-only (no torque control)
- Gripper: pneumatic, single-bit I/O via USB-6363 DAQ
