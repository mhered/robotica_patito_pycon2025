## Running inference to evaluate the policy

## ACT

```bash
(lerobot)$ lerobot-record \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower_arm \
  --robot.cameras="{ head_cam: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30}, top_cam: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 30} }" \
  --display_data=true \
  --dataset.repo_id=mhered/eval_recording-test \
  --dataset.single_task="Grab a duckie" \
  --policy.path=mhered/my_act \
  --dataset.num_episodes=2 \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader_arm
```



## SmolVLA

```bash
(lerobot)$ lerobot-record \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower_arm \
  --robot.cameras="{ head_cam: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30}, top_cam: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 30} }" \
  --display_data=true \
  --dataset.repo_id=mhered/eval_recording-test \
  --dataset.single_task="Grab a duckie" \
  --policy.path=mhered/my_smolvla \
  --dataset.num_episodes=1 \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader_arm
```


