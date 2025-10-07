## Quick Start

### Setup

1. Set up leader and follower arm, secure them with clamps to the table

2. Connect arms to power supply and switch it on

3. Connect to the laptop via USB the two arms. Order matters!! First the follower arm and then the leader arm means the follower will be assigned  `/dev/ttyACM0` and the leader `/dev/ttyACM1` . You may want to check ports live with

   ```bash
   $ watch -n 1 ls /dev/ttyACM*
   ```


### Basic teleop

1. Activate the conda environment and check teleoperation without cameras. This command will automatically
   1. Identify any missing calibrations and initiate the guided calibration procedure if needed.
   2. Start teleoperation.

```bash
$ conda activate lerobot
(lerobot)$ lerobot-teleoperate \
    --robot.type=so100_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_follower_arm \
    --teleop.type=so100_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_leader_arm
```

![](./assets/teleop.gif)

### Setup recording

1. Set up cameras (e.g. top camera and wrist camera), connect them to the laptop, then find their indexes running the following command and looking at the images stored in the `outputs/captured_images` folder. In our case `/dev/video4` is `head_cam` and `/dev/video6` is `top_cam`

```bash
(lerobot)$ lerobot-find-cameras opencv
```

### Start recording episodes

```bash
(lerobot)$ lerobot-record \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower_arm \
  --robot.cameras="{ head_cam: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30}, top_cam: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 30} }" \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader_arm \
  --display_data=true \
  --dataset.repo_id=${HF_USER}/recording-test \
  --dataset.num_episodes=3 \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=15 \
  --dataset.single_task="Grab a duckie"
```

Control the data recording flow using keyboard shortcuts:

- Press **Right Arrow (`→`)**: Early stop the current episode or reset time and move to the next.
- Press **Left Arrow (`←`)**: Cancel the current episode and re-record it.
- Press **Escape (`ESC`)**: Immediately stop the session, encode videos, and upload the dataset.

### Resume recording more episodes

```bash
(lerobot)$ lerobot-record \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower_arm \
  --robot.cameras="{ head_cam: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30}, top_cam: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 30} }" \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader_arm \
  --display_data=true \
  --dataset.repo_id=${HF_USER}/recording-test \
  --dataset.num_episodes=5 \
  --dataset.episode_time_s=80 \
  --dataset.reset_time_s=10 \
  --dataset.single_task="Grab a duckie" \
  --resume=true
```

### Advice for recording good episodes

- look only at the camera feeds: if it is difficult for you to complete the task looking only at camera feeds it will be difficult for the model
- keep leader arm, moving background out of the camera field of view
- Record 60 episodes
  * first ~20 in roughly the same position
  * ~20 changing starting positions of duckie and barrel
  * ~20 varying also the lighting and the color of the barrel to add robustness and avoid overfitting

### Visualize datasets

v2.0 stores episodes in an unfriendly format, but there is a visualizer:
https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fmhered%2Frecording-test

