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

For robot workflows plus training:

```bash
pip install -e ".[core_scripts,training]"
```

For the full local package with all available extras:

```bash
pip install -e ".[all]"
```

The editable `-e` install keeps the installed package connected to this local
checkout, so code changes and future pulls are reflected in the environment.

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

## Updating After Pulling Changes

```bash
cd /path/to/lerobot
git pull
conda activate lerobot
pip install -e ".[core_scripts,training]"
```

If dependency files changed, reinstall the relevant extras with the same
`pip install -e ...` command you used originally.

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

If a Python package build fails, make sure the Ubuntu packages from the
installation section are installed, then rerun the relevant `pip install -e ...`
command.

## License and Attribution

This repository is based on Hugging Face LeRobot, which is licensed under the
Apache License 2.0. The upstream license and copyright notice are preserved in
`LICENSE`.

When publishing or sharing this fork, keep the `LICENSE` file and this upstream
attribution. Personal changes in this fork are for study and experimentation
unless stated otherwise.
