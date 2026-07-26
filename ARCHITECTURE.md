# Architecture · Vision-Guided 6-DOF Robotic Arm Sorting

## 1. Pipeline Timing

```
t=0 ms    Camera frame arrives (RGB + Depth, 30 fps)
t=5 ms    YOLOv5 inference (640×640 input, GPU)
t=15 ms   Hand-eye transform: pixel → base frame
t=20 ms   Priority queue update + target selection
t=25 ms   MoveIt RRTConnect plan (max 200 ms budget)
t=225 ms  Trajectory smoothing + safety check
t=235 ms  Joint trajectory dispatch (50 Hz control)
t≈7 s     Gripper close + return-to-home + bin drop
```

The detection and planning loops are decoupled; control runs at 50 Hz on its
own thread.

## 2. State Machine

```
IDLE ──[new frame]──> DETECTING ──[target found]──> PLANNING
  ▲                       │                            │
  │                  [no target]                  [plan failed]
  │                       ▼                            │
  │                    IDLE ◄─────── SKIP ◄────────────┤
  │                                                  │
  └─────────────────── RETURNING ◄────────────── EXECUTING
                            │                          │
                            └──[gripper closed]────────┘
```

## 3. Failure Recovery

- **Detection miss** → wait for next frame (no retries, skip and continue)
- **Plan failure** (collision) → re-pose target, try once, otherwise skip
- **Grasp miss** (gripper closed but no contact) → reopen, retry target 1×,
  otherwise skip and continue
- **Grasp drop** (target slips during move) → drop event into Bin A as
  "unknown", continue

## 4. Topics (ROS Noetic)

| Topic | Type | Rate | Producer → Consumer |
|---|---|---|---|
| `/camera/rgb/image_raw` | sensor_msgs/Image | 30 Hz | realsense2_camera → yolo_detector |
| `/camera/depth/image_raw` | sensor_msgs/Image | 30 Hz | realsense2_camera → hand_eye_node |
| `/detections` | custom_msgs/Detection | 10 Hz | yolo_detector → priority_queue |
| `/target_pose` | geometry_msgs/Pose | on demand | priority_queue → moveit_planner |
| `/joint_trajectory` | trajectory_msgs/JointTrajectory | 50 Hz | moveit_planner → arm_controller |
| `/gripper/cmd` | std_msgs/Bool | on demand | state_machine → gripper_driver |
