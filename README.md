# Personal VLA Robot Learning Study Repository

This repository is my personal research and learning fork based on Hugging Face
LeRobot. I use it to study state-of-the-art Vision-Language-Action (VLA) robot
learning, robot policy training, dataset workflows, and real-world robotics
tooling.

Original upstream project: https://github.com/huggingface/lerobot

This is not an official Hugging Face project. It is a personal study repository
for experiments, notes, and reproducible robotics learning work.

## Focus Areas

- Vision-Language-Action robot learning
- Policy training and evaluation
- LeRobot dataset recording and replay
- Robot control workflows
- Simulation and real hardware experimentation
- Reproducible installation on Ubuntu 22.04

## Repository Contents

- `src/lerobot/`: Python package source code
- `examples/`: example training, dataset, and robot workflows
- `docs/`: reference documentation from the upstream project
- `tests/`: package tests
- `pyproject.toml`: Python package metadata and dependency extras
- `requirements-ubuntu.txt`: pinned Ubuntu dependency set
- `LICENSE`: upstream Apache-2.0 license notice

## Ubuntu 22.04 Installation

LeRobot requires Python 3.12 or newer. Ubuntu 22.04 usually ships with Python
3.10, so a dedicated conda environment is recommended.

### 1. Install system packages

```bash
sudo apt update
sudo apt install -y \
  git \
  wget \
  build-essential \
  cmake \
  pkg-config \
  python3-dev \
  ffmpeg \
  libavformat-dev \
  libavcodec-dev \
  libavdevice-dev \
  libavutil-dev \
  libswscale-dev \
  libswresample-dev \
  libavfilter-dev \
  libevdev-dev
```

For serial robot hardware, add your user to the `dialout` group:

```bash
sudo usermod -aG dialout "$USER" example user system name
```

Log out and log back in after changing groups.

### 2. Install Miniforge

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

Close and reopen the terminal, or run:

```bash
source ~/miniforge3/etc/profile.d/conda.sh
```

### 3. Clone the repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd lerobot
```

If the repository already exists locally:

```bash
cd /path/to/lerobot
git pull
```

### 4. Create and activate the environment

```bash
conda create -y -n lerobot python=3.12
conda activate lerobot
python -m pip install --upgrade pip setuptools wheel
conda install -y ffmpeg -c conda-forge
```

Activate this environment in every new terminal before using the package:

```bash
conda activate lerobot
```

### 5. Install the package

For robot workflows such as calibration, teleoperation, recording, replay, and
visualization:

```bash
pip install -e ".[core_scripts]"
```

For training workflows:

```bash
pip install -e ".[training]"
```
# LeRobot Personal Study Repository

This repository is a personal research and learning fork based on Hugging Face LeRobot. It is used to study Vision-Language-Action robot learning, policy training, dataset workflows, and real-world robotics tooling.

Original upstream project: https://github.com/huggingface/lerobot

This is not an official Hugging Face project. It is a personal study repository for experiments, notes, and reproducible robotics learning work.

## Overview

- Vision-Language-Action robot learning
- Policy training and evaluation
- LeRobot dataset recording and replay
- Robot control workflows
- Simulation and real hardware experimentation
- Reproducible installation on Ubuntu 22.04

## Repository Layout

- src/lerobot/: Python package source code
- examples/: example training, dataset, and robot workflows
- docs/: reference documentation from the upstream project
- tests/: package tests
- pyproject.toml: Python package metadata and dependency extras
- requirements-ubuntu.txt: pinned Ubuntu dependency set
- LICENSE: upstream Apache-2.0 license notice

## Requirements

- Ubuntu 22.04
- Python 3.12 or newer
- ffmpeg for video decoding and recording
- conda or Miniforge recommended for environment management

## Installation on Ubuntu 22.04

Ubuntu 22.04 usually ships with Python 3.10, so a dedicated conda environment is recommended.

### 1. Install system packages

```bash
sudo apt update
sudo apt install -y \
  git \
  wget \
  build-essential \
  cmake \
  pkg-config \
  python3-dev \
  ffmpeg \
  libavformat-dev \
  libavcodec-dev \
  libavdevice-dev \
  libavutil-dev \
  libswscale-dev \
  libswresample-dev \
  libavfilter-dev \
  libevdev-dev
```

For serial robot hardware, add your user to the dialout group:

```bash
sudo usermod -aG dialout "$USER"
```

Log out and log back in after changing groups.

### 2. Install Miniforge

```bash
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

Close and reopen the terminal, or run:

```bash
source ~/miniforge3/etc/profile.d/conda.sh
```

### 3. Clone the repository

```bash
git clone <YOUR_REPOSITORY_URL>
cd Lerobot-Robot-Learning-Lab
```

If the repository already exists locally:

```bash
cd /path/to/Lerobot-Robot-Learning-Lab
git pull
```

### 4. Create and activate the environment

```bash
conda create -y -n lerobot python=3.12
conda activate lerobot
python -m pip install --upgrade pip setuptools wheel
conda install -y ffmpeg -c conda-forge
```

Activate this environment in every new terminal before using the package:

```bash
conda activate lerobot
```

### 5. Install the package

For robot workflows such as calibration, teleoperation, recording, replay, and visualization:

```bash
pip install -e ".[core_scripts]"
```

For training workflows:

```bash
pip install -e ".[training]"
```

For robot workflows plus training:

```bash
pip install -e ".[core_scripts,training]"
```

For the full local package with all available extras:

```bash
pip install -e ".[all]"
```

The editable `-e` install keeps the package connected to this local checkout, so code changes and future pulls are reflected in the environment.

## Verification

```bash
conda activate lerobot
lerobot-info
python -c "import lerobot; print(lerobot.__version__)"
```

Useful hardware checks:

```bash
lerobot-find-cameras
lerobot-find-port
```

## Common Commands

```bash
lerobot-calibrate --help
lerobot-teleoperate --help
lerobot-record --help
lerobot-replay --help
lerobot-train --help
lerobot-eval --help
```

## SO-101 Teleoperation

Use this command to teleoperate an SO-101 follower arm from an SO-101 leader arm. Activate the environment first, then pass the serial ports for each arm:

```bash
conda activate lerobot
lerobot-teleoperate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader
```

If your devices appear on different ports, run lerobot-find-port and replace /dev/ttyACM1 and /dev/ttyACM0 with the detected follower and leader ports.

## Optional Hardware Extras

Install only the hardware extras needed for a specific setup:

```bash
pip install -e ".[feetech]"
pip install -e ".[dynamixel]"
pip install -e ".[intelrealsense]"
pip install -e ".[gamepad]"
pip install -e ".[lekiwi]"
pip install -e ".[reachy2]"
```

Multiple extras can be combined:

```bash
pip install -e ".[feetech,intelrealsense,gamepad]"
```

## Behavioral Cloning with ZED

The ZED camera can be used as a standard OpenCV/V4L2 webcam for recording behavioral cloning datasets.

Detected cameras on this machine:

- /dev/video0: HP laptop webcam
- /dev/video2: ZED camera working stream
- /dev/video3: ZED secondary stream, not used

Quick test:

```bash
ffplay /dev/video2
```

### Example recording command

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower \
  --robot.cameras="{ front: {type: opencv, index_or_path: 2, width: 1334, height: 480, fps: 30} }" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader \
  --dataset.repo_id=<hf_user>/<dataset_name> \
  --dataset.num_episodes=10 \
  --dataset.single_task="Pick and place the object" \
  --dataset.streaming_encoding=true \
  --dataset.encoder_threads=2 \
  --display_data=true
```

### Example training command

```bash
lerobot-train \
  --dataset.repo_id=<hf_user>/<dataset_name> \
  --policy.type=act \
  --output_dir=outputs/train/bc_zed \
  --job_name=bc_zed \
  --policy.device=cuda
```

If a Python package build fails, make sure the Ubuntu packages from the installation section are installed, then rerun the relevant `pip install -e ...` command.

## Updating After Pulling Changes

```bash
cd /path/to/Lerobot-Robot-Learning-Lab
git pull
conda activate lerobot
pip install -e ".[core_scripts,training]"
```

If dependency files changed, reinstall the relevant extras with the same `pip install -e ...` command used originally.

## Troubleshooting

If `conda activate lerobot` fails:

```bash
~/miniforge3/bin/conda init bash
source ~/.bashrc
conda activate lerobot
```

If serial devices are not accessible:

```bash
groups
ls -l /dev/ttyUSB* /dev/ttyACM*
```

If video decoding or recording fails:

```bash
ffmpeg -version
```

## SO-101 + ZED Behavioral Cloning Recording Protocol

### Current setup

- Robot: SO-101 leader-follower
- Follower arm: /dev/ttyACM0
- Leader arm: /dev/ttyACM1
- Camera: ZED camera
- Camera stream: /dev/video2
- Dataset repo: devadasvijayansheela/so101_pick_place_cube
- Task: Pick cube and place into tape box / target square

### 1. Check devices

```bash
ls /dev/ttyACM*
v4l2-ctl --list-devices
```

Expected devices:

```text
/dev/ttyACM0
/dev/ttyACM1

ZED 2:
  /dev/video2
  /dev/video3
```

### 2. Activate environment

```bash
conda activate lerobot
cd ~/Lerobot-Robot-Learning-Lab
```

### 3. Start recording

```bash
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower \
  --robot.cameras="{ front: {type: opencv, index_or_path: '/dev/video2', width: 1344, height: 376, fps: 30} }" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader \
  --dataset.repo_id=devadasvijayansheela/so101_pick_place_cube \
  --dataset.num_episodes=20 \
  --dataset.single_task="Pick cube and place into tape box" \
  --display_data=true
```

### 4. Correct episode procedure

For every episode:

1. Put the cube in the start position.
2. Put the target box or tape square in the target position.
3. Put the robot arm in the home pose.
4. Press Enter when the terminal asks to start.
5. Move the leader arm smoothly.
6. Pick up the cube.
7. Lift the cube.
8. Move to the target.
9. Release the cube.
10. Move the arm slightly away.
11. Press Enter when the terminal asks to stop or finish.

### 5. If an episode fails

Examples of failed episodes:

- Missed cube
- Bad grasp
- Cube dropped
- Robot hits object
- Camera blocked
- Cube not placed correctly

For the first dataset, failed demonstrations should not be kept.

If failure happens during recording:

```bash
Ctrl + C
```

Then restart the recording command and record again.

Goal:

- 20 successful episodes, not just 20 attempts

Example tracking:

- Episode 1: success
- Episode 2: success
- Episode 3: failed grasp, discard
- Episode 3: re-record success
- Episode 4: success
- Episode 20: success

### 6. Where failed recordings are saved

LeRobot saves local datasets under:

```text
~/.cache/huggingface/lerobot/devadasvijayansheela/
```

Check saved files:

```bash
find ~/.cache/huggingface/lerobot/devadasvijayansheela -name "*.mp4"
find ~/.cache/huggingface/lerobot/devadasvijayansheela -name "*.parquet"
```

If you cancel with Ctrl + C, partial or failed data may remain in a timestamped folder.

Example:

```text
~/.cache/huggingface/lerobot/devadasvijayansheela/so101_pick_place_cube_20260617_000501/
```

To remove a bad local dataset folder:

```bash
rm -rf ~/.cache/huggingface/lerobot/devadasvijayansheela/<bad_dataset_folder_name>
```

Only delete folders you are sure are bad.

### 7. Cube position rule

For the first 20 episodes:

- Keep cube position almost identical
- Keep target box or tape square fixed
- Keep camera fixed
- Keep lighting fixed

Reason:

The first policy should learn the basic pick-and-place motion.

After the first policy works, record more varied data:

- Episodes 21–50: small cube position variation
- Episodes 51–100: larger cube variation and different cube orientations

### 8. After recording 20 successful episodes

Check local dataset:

```bash
find ~/.cache/huggingface/lerobot/devadasvijayansheela -name "*.mp4"
find ~/.cache/huggingface/lerobot/devadasvijayansheela -name "*.parquet"
```

Then train:

```bash
lerobot-train \
  --dataset.repo_id=devadasvijayansheela/so101_pick_place_cube \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_pick_place_cube \
  --job_name=act_so101_pick_place_cube \
  --policy.device=cuda \
  --wandb.enable=false
```

### 9. Dataset size guide

- 1 episode: pipeline test
- 20 episodes: first small dataset
- 50 episodes: first useful training
- 100+ episodes: better policy
- 200+ episodes: more reliable pick-and-place

### 10. Main rule

For behavioral cloning:

- Good demonstrations teach good behavior
- Failed demonstrations teach confusion

For the first dataset, keep only clean successful pick-and-place demonstrations.

## License and Attribution

This repository is based on Hugging Face LeRobot, which is licensed under the Apache License 2.0. The upstream license and copyright notice are preserved in LICENSE.

When publishing or sharing this fork, keep the LICENSE file and this upstream attribution. Personal changes in this fork are for study and experimentation unless stated otherwise.