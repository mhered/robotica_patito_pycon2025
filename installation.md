# First Setup

## Setup conda

Sources: https://huggingface.co/docs/lerobot/en/installation and https://huggingface.co/docs/lerobot/so100

install `miniconda`:

```bash
$ mkdir -p ~/miniconda3
$ wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
$ bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
$ rm ~/miniconda3/miniconda.sh
```

source and initialize `conda`:

```bash
$ source ~/miniconda3/bin/activate
$ conda init --all
```

to prevent conda for launching automatically:

```bash
$ conda config --set auto_activate_base false
```

create a virtual environment with python 3.10, activate it and install `ffmpeg`:

```bash
$ conda create -y -n lerobot python=3.10
$ conda activate lerobot
(lerobot)$ conda install ffmpeg -c conda-forge
```

## Install lerobot (from source) 

Clone the repo:

```bash
(lerobot)$ git clone https://github.com/huggingface/lerobot.git
(lerobot)$ cd lerobot
```

install dependencies and lerobot:

```bash
(lerobot)$ sudo apt-get install cmake build-essential python3-dev pkg-config libavformat-dev libavcodec-dev libavdevice-dev libavutil-dev libswscale-dev libswresample-dev libavfilter-dev pkg-config
(lerobot)$ pip install -e ".[feetech]"
```

##  Connect the arms

Connect follower first then leader arm while checking ports live with

```bash
(lerobot)$ watch -n 1 ls /dev/ttyACM*
```

## Calibrate

### Follower arm

```bash
(lerobot)$ lerobot-calibrate \
    --robot.type=so100_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_follower_arm
    
INFO 2025-09-16 23:02:01 calibrate.py:73 {'robot': {'calibration_dir': None,
           'cameras': {},
           'disable_torque_on_disconnect': True,
           'id': 'my_follower_arm',
           'max_relative_target': None,
           'port': '/dev/ttyACM0',
           'use_degrees': False},
 'teleop': None}
INFO 2025-09-16 23:02:01 follower.py:104 my_follower_arm SO100Follower connected.
INFO 2025-09-16 23:02:01 follower.py:121 
Running calibration of my_follower_arm SO100Follower
Move my_follower_arm SO100Follower to the middle of its range of motion and press ENTER....
Move all joints except 'wrist_roll' sequentially through their entire ranges of motion.
Recording positions. Press ENTER to stop...

-------------------------------------------
-------------------------------------------
NAME            |    MIN |    POS |    MAX
shoulder_pan    |    651 |   2016 |   3162
shoulder_lift   |    911 |    912 |   3225
elbow_flex      |    831 |   3063 |   3073
wrist_flex      |    834 |   2907 |   3288
gripper         |   2044 |   2056 |   3487
Calibration saved to /home/mhered/.cache/huggingface/lerobot/calibration/robots/so100_follower/my_follower_arm.json
INFO 2025-09-16 23:03:06 follower.py:232 my_follower_arm SO100Follower disconnected.
```

### Leader arm

```bash
(lerobot)$ lerobot-calibrate \
    --teleop.type=so100_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_leader_arm
   
INFO 2025-09-16 23:07:18 calibrate.py:73 {'robot': None,
 'teleop': {'calibration_dir': None,
            'id': 'my_leader_arm',
            'port': '/dev/ttyACM1'}}
INFO 2025-09-16 23:07:18 00_leader.py:81 my_leader_arm SO100Leader connected.
INFO 2025-09-16 23:07:18 00_leader.py:98 
Running calibration of my_leader_arm SO100Leader
Move my_leader_arm SO100Leader to the middle of its range of motion and press ENTER....
Move all joints except 'wrist_roll' sequentially through their entire ranges of motion.
Recording positions. Press ENTER to stop...

-------------------------------------------
-------------------------------------------
NAME            |    MIN |    POS |    MAX
shoulder_pan    |    881 |   1837 |   3204
shoulder_lift   |    975 |    976 |   3269
elbow_flex      |    803 |   3015 |   3015
wrist_flex      |    652 |   2822 |   3116
gripper         |   2043 |   2046 |   2959
Calibration saved to /home/mhered/.cache/huggingface/lerobot/calibration/teleoperators/so100_leader/my_leader_arm.json
INFO 2025-09-16 23:08:20 0_leader.py:159 my_leader_arm SO100Leader disconnected.
```

Note calibration data is now saved to:

`/home/mhered/.cache/huggingface/lerobot/calibration/robots/so100_follower/my_follower_arm.json`

`/home/mhered/.cache/huggingface/lerobot/calibration/teleoperators/so100_leader/my_leader_arm.json`

## Teleoperate

```bash
(lerobot)$ lerobot-teleoperate \
    --robot.type=so100_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_follower_arm \
    --teleop.type=so100_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_leader_arm
```

This command will automatically:

1. Identify any missing calibrations and initiate the calibration procedure.
2. Connect the robot and teleop device and start teleoperation.

## Setup cameras

Source: https://huggingface.co/docs/lerobot/en/cameras

Find camera indexes with:

```bash
(lerobot)$ lerobot-find-cameras opencv

--- Detected Cameras ---
Camera #0:
  Name: OpenCV Camera @ /dev/video0
  Type: OpenCV
  Id: /dev/video0
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Width: 640
    Height: 480
    Fps: 30.0
--------------------
Camera #1:
  Name: OpenCV Camera @ /dev/video2
  Type: OpenCV
  Id: /dev/video2
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Width: 640
    Height: 360
    Fps: 40.0
--------------------
Camera #2:
  Name: OpenCV Camera @ /dev/video4
  Type: OpenCV
  Id: /dev/video4
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Width: 640
    Height: 480
    Fps: 30.0
--------------------
Camera #3:
  Name: OpenCV Camera @ /dev/video6
  Type: OpenCV
  Id: /dev/video6
  Backend api: V4L2
  Default stream profile:
    Format: 0.0
    Width: 640
    Height: 480
    Fps: 30.0
--------------------

Finalizing image saving...
Image capture finished. Images saved to outputs/captured_images

```

Identify cameras by looking at the images stored in the `outputs/captured_images` folder

In this case `/dev/video4` is `head_cam` and `/dev/video6` is `top_cam`

## Teleop (& visualize) with cameras 

```bash
(lerobot)$ lerobot-teleoperate \
    --robot.type=so100_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_follower_arm \
    --robot.cameras="{ head_cam: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30}, top_cam: {type: opencv, index_or_path: 6, width: 640, height: 480, fps: 30}}" \
    --teleop.type=so100_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_leader_arm \
    --display_data=true
```

## Setup HuggingFace token 

Source: https://huggingface.co/docs/lerobot/en/il_robots

get a token, add to the repo credential, and store HF repo name in a variable:

```bash
(lerobot)$ hf auth login --token hf_**...** --add-to-git-credential
Token is valid (permission: fineGrained).
The token `vader-pycon` has been saved to /home/mhered/.cache/huggingface/stored_tokens
Your token has been saved in your configured git credential helpers (store).
Your token has been saved to /home/mhered/.cache/huggingface/token
Login successful.
The current active token is: `vader-pycon`

(lerobot)$ HF_USER=$(hf auth whoami | awk '{print $NF}') # awk used to extract last field of command output
(lerobot)$ echo $HF_USER
mhered
```

## Record a couple of episodes (init repo)

Source: https://huggingface.co/docs/lerobot/en/il_robots

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

## Add more episodes (resume recording)

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

Advice:

- look only at the camera feeds: if it is difficult for you to complete the task looking only at camera feeds it will be difficult for the model
- keep leader arm, moving background out of the camera field of view
- Record 60 episodes
  * first ~20 in roughly the same position
  * ~20 changing starting positions of duckie and barrel
  * ~20 varying also the lighting and the color of the barrel to add robustness and avoid overfitting

## Visualize datasets

v2.0 stores episodes in an unfriendly format, but there is a visualizer:
https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fmhered%2Frecording-test