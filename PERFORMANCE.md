# Performance · Test Methodology and Results

## 1. Test Methodology

### 1.1 Offline (YOLOv5 detection)

- **Test set**: 500 images, captured on the production line during a
  representative shift, never used in training
- **Metric**: mean Average Precision at IoU=0.5 (mAP@0.5)
- **Result**: **mAP@0.5 = 96.2 %**

### 1.2 Online (system-level pick success)

- **Definition**: A *pick* is successful when the gripper closes on a target
  and the target ends up in the correct bin within the cycle time budget.
- **Test**: 200 picks over an 8-hour continuous run, mixed parts, ambient
  lighting, normal conveyor speed
- **Result**: **198/200 = 99 %** single-attempt, **98 %** with retry counted
  as success-or-skip

### 1.3 Cycle Time

- **Definition**: Time from "frame in" to "gripper open in bin"
- **Test**: 50 consecutive picks, median over the run
- **Result**: **7.0 s** (p50), 7.8 s (p95), 9.1 s (p99)

## 2. Failure Analysis

The 2 % failure rate decomposed as:

| Failure mode | Frequency | Mitigation in v1.1 |
|---|---|---|
| Multi-target occlusion | 1.1 % | Coarse-to-fine re-pose when distance < 50 px |
| Specular reflection on depth | 0.6 % | Polarizing filter, hand-tuned exposure |
| Pneumatic gripper slip | 0.3 % | Increase grip force on metal parts only |

## 3. Uptime

8-hour shift, 0 unplanned stops, 0 safety trips, 1 planned pause (lunch).

## 4. Comparison vs. Manual Pick

| | Manual | System (v1.0) | Notes |
|---|---|---|---|
| Throughput (picks/h) | ~400 | ~510 | system includes 1 operator for tray refill |
| Pick accuracy | ~99 % | 98 % | within spec |
| Operator fatigue cost | high | low | night-shift no longer required |
