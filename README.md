# LeRobot SO-101 Robot Learning Lab

This repository is a study-purpose clone/fork of Hugging Face LeRobot, organized as a practical robot-learning lab for the SO-101 leader-follower platform. It documents the full workflow from teleoperated demonstrations to ACT policy training and real-robot rollout.

Original upstream project: [https://github.com/huggingface/lerobot](https://github.com/huggingface/lerobot)

Hugging Face LeRobot: [https://huggingface.co/lerobot](https://huggingface.co/lerobot)

## Project Goal

The goal is to train a robot policy that learns from human demonstrations. A human moves the SO-101 leader arm by hand, while the SO-101 follower arm mirrors the motion. During this process, LeRobot records camera images, robot joint states, actions, timestamps, episode indices, and the task instruction. The recorded dataset is then used to train an imitation-learning policy for autonomous robot control.

Task:

```text
Pick black cube and place into square box
```

In one sentence:

```text
Teleoperation demonstrations -> LeRobot dataset -> ACT training -> checkpoint -> SO-101 rollout
```

## Demo Overview

<table>
<tr>
<td align="center" width="50%">
<b>Step 1: Human Teleoperation</b><br><br>
<a href="assets/gif/human3.gif"><img src="assets/gif/human3.gif" height="220"></a>
<br><br>
<sub>The operator moves the leader arm. The follower arm mirrors the motion in real time, producing expert demonstrations.</sub>
</td>
<td align="center" width="50%">
<b>Step 2: Rerun Monitoring</b><br><br>
<a href="assets/gif/rerun3.gif"><img src="assets/gif/rerun3.gif" height="220"></a>
<br><br>
<sub>Rerun visualizes camera frames, robot state, actions, trajectories, and live debugging data during recording.</sub>
</td>
</tr>
<tr>
<td align="center" width="50%">
<b>Step 3: Dataset Visualization</b><br><br>
<a href="assets/gif/datavisulisation3.gif"><img src="assets/gif/datavisulisation3.gif" height="220"></a>
<br><br>
<sub>The recorded LeRobot dataset is inspected before training to verify image quality, actions, and trajectories.</sub>
</td>
<td align="center" width="50%">
<b>Step 4: ACT Policy Deployment</b><br><br>
<a href="assets/gif/result3.gif"><img src="assets/gif/result3.gif" height="220"></a>
<br><br>
<sub>The trained ACT checkpoint is loaded for real-robot rollout on the SO-101 follower arm.</sub>
</td>
</tr>
</table>

## Experiment Summary

| Item | Value |
| --- | --- |
| Robot | LeRobot SO-101 leader-follower setup |
| Follower arm | `so101_follower` |
| Leader arm | `so101_leader` |
| Follower port | `/dev/ttyACM0` |
| Leader port | `/dev/ttyACM1` |
| Camera | OpenCV camera at `/dev/video2` |
| Camera resolution | 1344 x 376 |
| FPS | 30 |
| OS | Ubuntu 22.04 |
| Environment | Conda environment named `lerobot` |
| GPU | NVIDIA RTX 3050, 6 GB |
| LeRobot version | 0.5.2 |
| Policy | ACT, Action Chunking Transformer |
| Dataset | `devadasvijayansheela/so101_pick_place_cube_20260617_014712` |
| Dataset size | 20 episodes, 17,893 frames |
| Training output | `outputs/train/act_so101_pick_place_cube_quick` |

## Repository Role

This GitHub repository should contain code, documentation, configuration, and small demo assets. Large runtime artifacts should stay out of Git.

Keep these ignored:

```text
outputs/
data/
.cache/
wandb/
```

Use Hugging Face Hub for large artifacts:

```text
GitHub: code, docs, commands, small GIFs
Hugging Face: datasets, trained policies, checkpoints
```

## End-to-End Pipeline

```text
1. Connect SO-101 leader and follower arms
2. Calibrate robot arms
3. Connect OpenCV camera
4. Teleoperate follower arm using leader arm
5. Record demonstration episodes
6. Upload dataset to Hugging Face
7. Train ACT imitation-learning policy
8. Save checkpoints
9. Deploy checkpoint to hardware
10. Evaluate autonomous pick-and-place behavior
```

System flow:

```text
Human teleoperation
        ↓
Dataset recording
        ↓
Dataset inspection
        ↓
ACT training
        ↓
Checkpoint generation
        ↓
Real-robot rollout
        ↓
Evaluation and iteration
```

## 1. Activate Environment

```bash
conda activate lerobot
cd ~/Lerobot-Robot-Learning-Lab
```

For a fresh development setup with `uv`:

```bash
uv sync --locked --extra feetech --extra training --extra core_scripts
```

For testing and code review:

```bash
uv sync --locked --extra test --extra dev
```

## 2. Check Connected Devices

Find robot ports:

```bash
lerobot-find-port
```

Expected robot ports for this setup:

```text
/dev/ttyACM0 -> SO-101 follower
/dev/ttyACM1 -> SO-101 leader
```

Find cameras:

```bash
lerobot-find-cameras
```

Detected camera used for recording:

```text
OpenCV camera: /dev/video2
Resolution: 1344 x 376
FPS: 30
```

## 3. Teleoperation Sanity Check

Before recording data, verify that the leader-follower control loop works.

```bash
lerobot-teleoperate \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower \
  --robot.cameras="{ front: {type: opencv, index_or_path: '/dev/video2', width: 1344, height: 376, fps: 30} }" \
  --teleop.type=so101_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=my_leader \
  --display_data=true
```

Check that:

- The follower arm mirrors the leader arm smoothly.
- The camera stream is visible in Rerun.
- The cube and box are visible from the camera viewpoint.
- There are no motor timeout or camera FPS errors.

## 4. Record Demonstrations

This command records 20 teleoperated demonstrations.

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
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=15 \
  --dataset.single_task="Pick black cube and place into square box" \
  --dataset.streaming_encoding=true \
  --dataset.encoder_threads=2 \
  --display_data=true
```

Important arguments:

| Argument | Purpose |
| --- | --- |
| `--robot.type=so101_follower` | Physical follower arm that executes motion |
| `--robot.port=/dev/ttyACM0` | Serial port of the follower arm |
| `--robot.id=my_follower` | Calibration ID for follower arm |
| `--robot.cameras=...` | Camera observation used by the policy |
| `--teleop.type=so101_leader` | Leader arm used for human teleoperation |
| `--teleop.port=/dev/ttyACM1` | Serial port of the leader arm |
| `--teleop.id=my_leader` | Calibration ID for leader arm |
| `--dataset.repo_id=...` | Hugging Face dataset repository |
| `--dataset.num_episodes=20` | Number of demonstrations |
| `--dataset.episode_time_s=30` | Duration of each episode |
| `--dataset.reset_time_s=15` | Time to reset cube, box, and robot |
| `--dataset.single_task=...` | Language instruction stored with the dataset |
| `--dataset.streaming_encoding=true` | Encodes video while recording |
| `--dataset.encoder_threads=2` | CPU threads used for video encoding |
| `--display_data=true` | Opens Rerun visualization |

Expected recording flow:

```text
Recording episode 0
Reset the environment
Recording episode 1
...
Stop recording
Processing Files
New Data Upload
Exiting
```

Useful controls:

| Control | Action |
| --- | --- |
| Right arrow | Finish current episode and continue |
| Left arrow | Discard and rerecord current episode |
| Escape | Stop recording and finalize dataset |

## 5. Dataset Result

Final dataset:

```text
Repository: devadasvijayansheela/so101_pick_place_cube_20260617_014712
Episodes: 20
Frames: 17,893
FPS: 30
Task: Pick black cube and place into square box
```

The dataset contains:

```text
Camera images
Robot joint states
Robot actions
Timestamps
Frame indices
Episode indices
Language instruction
```

Local dataset cache:

```bash
~/.cache/huggingface/lerobot/devadasvijayansheela/so101_pick_place_cube_20260617_014712
```

Find recorded videos:

```bash
find ~/.cache/huggingface/lerobot/devadasvijayansheela/so101_pick_place_cube_20260617_014712 -name "*.mp4"
```

Play a recorded video:

```bash
vlc ~/.cache/huggingface/lerobot/devadasvijayansheela/so101_pick_place_cube_20260617_014712/videos/observation.images.front/chunk-000/file-000.mp4
```

Before training, inspect:

- Camera blur or missing frames.
- Object visibility.
- Failed or hesitant demonstrations.
- Whether the cube and box positions match the intended task.
- Whether the demonstrations use a consistent strategy.

### Hugging Face Dataset Visualizer

The dataset can be inspected in the LeRobot Hugging Face dataset visualizer:

[View episode 0 in the LeRobot Dataset Visualizer](https://huggingface.co/spaces/lerobot/visualize_dataset?path=%2Fdevadasvijayansheela%2Fso101_pick_place_cube_20260617_014712%2Fepisode_0)

This view is useful for checking the same signals used during training:

- Front camera observations.
- Episode timing.
- Robot state and action trajectories.
- Whether the cube, box, and arm are visible throughout the episode.
- Whether the demonstration is clean enough to train from.

For this project, the visualizer is mainly a dataset-quality gate: if the episode looks inconsistent, blurred, incomplete, or visually ambiguous, it should be fixed before training more policies.

## 6. Project Artifacts and File Extensions

The files below are the most important project-specific artifacts. Large dataset and training artifacts should be referenced from Hugging Face or local paths, not committed directly to GitHub.

### Dataset Files

Video observations:

| Item | Value |
| --- | --- |
| Extension | `.mp4` |
| Local location | `~/.cache/huggingface/lerobot/devadasvijayansheela/so101_pick_place_cube_20260617_014712/videos/` |
| Example | `videos/observation.images.front/chunk-000/file-000.mp4` |
| Contains | Camera frames, visual observations, human demonstration videos |

Dataset tables:

| Item | Value |
| --- | --- |
| Extension | `.parquet` |
| Local examples | `data/chunk-000/file-000.parquet`, `meta/episodes/chunk-000/file-000.parquet`, `meta/tasks.parquet` |
| Contains | Joint positions, actions, timestamps, frame indices, episode information, task metadata |

Think of the `.parquet` files as the structured robot-learning dataset database, while the `.mp4` files store the visual observations.

Dataset metadata:

| Item | Value |
| --- | --- |
| Extension | `.json` |
| Local examples | `meta/info.json`, `meta/stats.json` |
| Contains | Dataset schema, feature statistics, normalization statistics, metadata used by LeRobot |

### Calibration Files

Calibration files are unique to this robot setup.

| Item | Value |
| --- | --- |
| Extension | `.json` |
| Follower location | `~/.cache/huggingface/lerobot/calibration/robots/so_follower/my_follower.json` |
| Leader location | `~/.cache/huggingface/lerobot/calibration/teleoperators/so_leader/my_leader.json` |
| Contains | Motor offsets, motor directions, joint calibration, robot-specific setup values |

These files should be preserved for your local robot, but they are machine/robot-specific and should be shared carefully.

### ACT Policy Files

The checkpoint used for deployment is:

```text
outputs/train/act_so101_pick_place_cube_quick/checkpoints/010000/
```

The most important trained-model file is:

```text
outputs/train/act_so101_pick_place_cube_quick/checkpoints/010000/pretrained_model/model.safetensors
```

| File | Extension | Meaning |
| --- | --- | --- |
| `model.safetensors` | `.safetensors` | The trained ACT neural network weights, effectively the robot policy brain |
| `config.json` | `.json` | ACT architecture and policy configuration, including features and model settings |
| `train_config.json` | `.json` | Training settings such as dataset, batch size, steps, save frequency, and optimizer settings |
| `policy_preprocessor.json` | `.json` | Policy input preprocessing pipeline definition |
| `policy_postprocessor.json` | `.json` | Policy output postprocessing pipeline definition |

If someone asks where the trained model is, the short answer is:

```text
model.safetensors
```

### Training State Files

Training state files are useful for resuming training, but they are not needed just to explain the project or run inference from an exported pretrained policy folder.

Location:

```text
outputs/train/act_so101_pick_place_cube_quick/checkpoints/010000/training_state/
```

| File | Extension | Purpose |
| --- | --- | --- |
| `training_step.json` | `.json` | Stores the current training step, for example step 10000 |
| `optimizer_param_groups.json` | `.json` | Stores optimizer parameter group metadata |
| `optimizer_state.safetensors` | `.safetensors` | Adam optimizer state, used when resuming training |
| `rng_state.safetensors` | `.safetensors` | Random number generator state for reproducible continuation |

### Normalization Files

Normalization is especially important in robot learning because joint values and actions must be scaled consistently between training and deployment.

| File | Extension | Purpose |
| --- | --- | --- |
| `policy_preprocessor_step_3_normalizer_processor.safetensors` | `.safetensors` | Normalizes observations and robot state before policy inference |
| `policy_postprocessor_step_0_unnormalizer_processor.safetensors` | `.safetensors` | Denormalizes policy outputs back into action values |

Without the matching normalizer and unnormalizer files, model predictions can be numerically wrong even if `model.safetensors` loads correctly.

### Hugging Face Policy Repository

When the policy is pushed to Hugging Face, the project policy repository is:

```text
devadasvijayansheela/act_so101_pick_place_cube_quick
```

The key uploaded policy files are typically:

```text
config.json
model.safetensors
train_config.json
policy_preprocessor.json
policy_postprocessor.json
normalizer / unnormalizer .safetensors files
```

### Project-Specific Artifact Hierarchy

```text
so101_pick_place_cube_project
|
|-- dataset
|   |-- data/*.parquet
|   |-- meta/*.json
|   |-- meta/**/*.parquet
|   `-- videos/**/*.mp4
|
|-- calibration
|   |-- my_follower.json
|   `-- my_leader.json
|
`-- ACT training checkpoint
    |-- pretrained_model/
    |   |-- model.safetensors
    |   |-- config.json
    |   |-- train_config.json
    |   |-- policy_preprocessor.json
    |   |-- policy_postprocessor.json
    |   |-- policy_preprocessor_step_3_normalizer_processor.safetensors
    |   `-- policy_postprocessor_step_0_unnormalizer_processor.safetensors
    |
    `-- training_state/
        |-- training_step.json
        |-- optimizer_param_groups.json
        |-- optimizer_state.safetensors
        `-- rng_state.safetensors
```

## 7. Train ACT

ACT stands for **Action Chunking Transformer**. For this project, it learns:

```text
camera image + robot joint state -> future robot action sequence
```

Training command:

```bash
lerobot-train \
  --dataset.repo_id=devadasvijayansheela/so101_pick_place_cube_20260617_014712 \
  --policy.type=act \
  --policy.repo_id=devadasvijayansheela/act_so101_pick_place_cube_quick \
  --output_dir=outputs/train/act_so101_pick_place_cube_quick \
  --job_name=act_so101_pick_place_cube_quick \
  --policy.device=cuda \
  --steps=30000 \
  --save_freq=5000 \
  --wandb.enable=false
```

Training parameters:

```text
Policy: ACT
Device: CUDA
GPU: RTX 3050 6 GB
Dataset frames: 17,893
Dataset episodes: 20
Batch size: 8
Training steps: 30,000
Checkpoint frequency: 5,000
WandB: disabled
```

Monitor GPU usage:

```bash
watch -n 1 nvidia-smi
```

Observed training behavior:

```text
GPU utilization: about 99% to 100%
GPU memory: about 4.8 GB to 5.0 GB / 6 GB
```

TensorBoard was tested:

```bash
tensorboard --logdir outputs/train --port 6006
```

This run did not produce TensorBoard event files:

```bash
find . -name "events.out.tfevents*"
```

For future runs, enable live experiment tracking with:

```bash
--wandb.enable=true
```

WandB can track loss, learning rate, gradient norm, GPU usage, training speed, and checkpoint history.

## 8. Checkpoints

Training output:

```text
outputs/train/act_so101_pick_place_cube_quick/
```

Checkpoint directory:

```text
outputs/train/act_so101_pick_place_cube_quick/checkpoints/
```

Example checkpoint:

```text
outputs/train/act_so101_pick_place_cube_quick/checkpoints/005000/
```

Important model file:

```text
outputs/train/act_so101_pick_place_cube_quick/checkpoints/005000/pretrained_model/model.safetensors
```

Check available checkpoints:

```bash
ls outputs/train/act_so101_pick_place_cube_quick/checkpoints
```

Observed/current deployment checkpoints:

```text
005000
010000
last
```

Planned or full 30k-step checkpoint sequence:

```text
005000
010000
015000
020000
025000
030000
```

Checkpoint used for deployment:

```text
010000
```

Policy location used:

```bash
outputs/train/act_so101_pick_place_cube_quick/checkpoints/010000/pretrained_model
```

## 9. Deployment and Rollout

In this repository version, `lerobot-record` is for teleoperated data collection. For trained-policy deployment, use `lerobot-rollout`.

### Initial Deployment Issues

| Attempt | Error | Reason |
| --- | --- | --- |
| `lerobot-record --policy.path=...` | `unrecognized arguments: --policy.path` | `lerobot-record` records demonstrations and does not execute trained policies in this repo version |
| `lerobot-rollout --policy.type=act --policy.pretrained_path=...` | `You must provide at least one image or the environment state among the inputs.` | ACT rollout required explicit feature definitions |
| `--policy.pretrained_path` only | `Expected a dict with a 'type' key` | ACT policy type must be specified |

### Working Deployment Command

This command successfully loaded the policy and started inference:

```bash
cd ~/Lerobot-Robot-Learning-Lab
conda activate lerobot

lerobot-rollout \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_follower \
  --robot.cameras="{ front: {type: opencv, index_or_path: '/dev/video2', width: 1344, height: 376, fps: 30} }" \
  --policy.type=act \
  --policy.pretrained_path=outputs/train/act_so101_pick_place_cube_quick/checkpoints/010000/pretrained_model \
  --policy.input_features="{ observation.state: {type: STATE, shape: [6]}, observation.images.front: {type: VISUAL, shape: [3, 376, 1344]} }" \
  --policy.output_features="{ action: {type: ACTION, shape: [6]} }" \
  --policy.device=cuda \
  --display_data=true
```

Observed output:

```text
Record loop is running slower than the target FPS
```

This indicates:

```text
Camera working
Policy loaded
ACT inference running
Robot connected
Real-world deployment started
```

### Camera Resolution Note

The policy was trained with image features shaped like:

```text
observation.images.front: [3, 376, 1344]
```

If you change the camera command to 640 x 480, the policy input feature shape must also match, and in practice the best result usually comes from training and rollout using the same camera resolution and viewpoint. Lower-resolution rollout should be treated as a separate experiment unless the policy/config supports that change.

## 10. Closed-Loop ACT Inference

During rollout, the robot does not execute one fixed script. It repeatedly observes the scene, predicts the next motor action, executes it, and observes again.

Example:

```text
Camera observation:
Cube at center
Box at right

ACT policy prediction:
Joint 1 = 0.52
Joint 2 = -0.18
Joint 3 = 0.73
...
```

These predicted joint commands are sent to the SO-101 motors. The robot moves, a new camera image arrives, the policy predicts again, and the loop repeats at approximately the control frequency, ideally close to 30 Hz.

Conceptually:

```text
Camera image + robot joint state
        ↓
ACT policy
        ↓
Predicted joint action
        ↓
Motor command
        ↓
Robot movement
        ↓
New camera image + new robot state
        ↓
Repeat
```

Rollout quality depends on both the trained policy and the real-time system:

- Camera FPS.
- Inference speed.
- Motor response.
- Demonstration quality.
- Calibration quality.
- Scene similarity between training and deployment.

## 11. Rollout Evaluation Metrics

For real-robot evaluation, measure staged success rates over repeated trials. A 10-trial evaluation is a simple first benchmark.

| Metric | Question | Example result |
| --- | --- | --- |
| Reach success | Can the robot reach the cube? | 8 / 10 reaches = 80% |
| Grasp success | Can it close the gripper correctly on the cube? | 7 / 10 grasps = 70% |
| Pick success | Can it lift the cube? | 6 / 10 lifts = 60% |
| Place success | Can it place the cube inside the box? | 5 / 10 placements = 50% |

The most important final metric for this task is usually **place success**, because it requires the full sequence: reaching, grasping, lifting, moving, and releasing into the target box.

Recommended rollout log:

| Trial | Checkpoint | Reach | Grasp | Pick | Place | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `010000` | yes | yes | yes | no | Dropped cube near box |
| 2 | `010000` | yes | no | no | no | Gripper closed too early |
| ... | ... | ... | ... | ... | ... | ... |

Failure interpretation:

| Pattern | Likely issue |
| --- | --- |
| High reach, low grasp | Gripper timing or grasp pose needs more demonstrations |
| Good grasp, poor lift | Grip strength, object slippage, or lift trajectory issue |
| Good lift, poor place | Box position variation or placement trajectory issue |
| Robot jitters | Noisy demonstrations, low FPS, or unstable action predictions |

## 12. Code Review Map

Use this map to review how the robot-learning system is implemented.

| Workflow stage | Main files | What to inspect |
| --- | --- | --- |
| CLI entry points | `pyproject.toml` | Maps commands like `lerobot-record`, `lerobot-train`, `lerobot-rollout`, `lerobot-teleoperate` |
| Teleoperation | `src/lerobot/scripts/lerobot_teleoperate.py` | Builds robot and teleoperator, reads leader actions, sends actions to follower |
| Recording | `src/lerobot/scripts/lerobot_record.py` | Records observations and actions into a LeRobot dataset |
| Dataset config | `src/lerobot/configs/dataset.py` | Episode count, FPS, reset time, video encoding, Hub repo settings |
| Dataset storage | `src/lerobot/datasets/lerobot_dataset.py` | Episode-aware data storage, videos, metadata, and frame access |
| Training | `src/lerobot/scripts/lerobot_train.py` | Dataset loading, policy creation, optimizer, training loop, checkpoints |
| Training config | `src/lerobot/configs/train.py` | Defaults for batch size, steps, checkpoint frequency, resume behavior |
| Policy factory | `src/lerobot/policies/factory.py` | Creates policies from `--policy.type` or checkpoint paths |
| Policy base | `src/lerobot/policies/pretrained.py` | Shared pretrained policy interface and Hub save/load behavior |
| ACT config | `src/lerobot/policies/act/configuration_act.py` | ACT hyperparameters, feature validation, optimizer preset |
| ACT model | `src/lerobot/policies/act/modeling_act.py` | Neural network, image/state inputs, action prediction |
| ACT processor | `src/lerobot/policies/act/processor_act.py` | ACT-specific preprocessing and postprocessing |
| SO-101 follower | `src/lerobot/robots/so_follower/` | Follower robot config and motor command implementation |
| SO-101 leader | `src/lerobot/teleoperators/so_leader/` | Leader-arm teleoperation implementation |
| OpenCV camera | `src/lerobot/cameras/opencv/` | USB camera discovery, connection, frame capture |
| Rollout | `src/lerobot/scripts/lerobot_rollout.py`, `src/lerobot/rollout/` | Real-robot policy deployment strategies |
| Tests | `tests/` | Policy, processor, dataset, teleoperator, and training utility tests |
| Examples | `examples/` | Training, teleoperation, evaluation, and tutorial scripts |
| Docs | `docs/source/` | Hardware, policy, dataset, and training documentation |

Useful code-review commands:

```bash
rg -n "lerobot-record|lerobot-train|lerobot-rollout|lerobot-teleoperate" pyproject.toml
find src/lerobot/policies/act -maxdepth 1 -type f -print
rg -n "def train|def update_policy|save_checkpoint|make_dataset|make_policy" src/lerobot/scripts/lerobot_train.py
rg -n "record_loop|robot.get_observation|teleop.get_action|robot.send_action|dataset.add_frame|save_episode" src/lerobot/scripts/lerobot_record.py
```

Focused tests for review:

```bash
uv run pytest tests/processor/test_act_processor.py tests/training/test_visual_validation.py -q
```

Full quality check:

```bash
uv run pre-commit run --all-files
```

## 13. Current Achievements

Completed:

- SO-101 leader-follower hardware setup.
- Leader-follower teleoperation.
- Camera integration through OpenCV.
- Rerun visualization during teleoperation and recording.
- Dataset recording: 20 demonstration episodes and 17,893 frames.
- Dataset upload to Hugging Face.
- ACT training on RTX 3050.
- Checkpoint generation.
- Real-robot policy loading through `lerobot-rollout`.
- End-to-end robot-learning demonstration from data collection to deployment.

## 14. Future Directions

These are the next logical steps after the current ACT-based SO-101 experiment.

### Direction 1: More Demonstrations

Current:

```text
20 episodes
17,893 frames
```

Future:

```text
100+ episodes
200+ episodes
500+ episodes
```

More demonstrations generally improve robustness, especially when they include object-position variation, recovery behavior, and consistent successful grasps.

### Direction 2: Multiple Objects

Current:

```text
One cube
One box
```

Future:

```text
Cube
Bottle
Marker
Cup
Toy
```

This would move the project from a single-object skill toward more general manipulation.

### Direction 3: Different Lighting and Backgrounds

Current:

```text
Single environment
Fixed lighting
Fixed background
```

Future:

```text
Bright room
Low-light room
Different backgrounds
Different object positions
```

This improves visual generalization and reduces overfitting to one room setup.

### Direction 4: ACT vs Other Policies

Current:

```text
ACT
```

Future comparison:

```text
ACT
Diffusion Policy
SmolVLA
Pi0
```

Compare:

```text
Success rate
Training time
Inference speed
GPU memory usage
Real-robot stability
```

### Direction 5: Sim-to-Real

Current:

```text
Real demonstrations
        ↓
Real robot deployment
```

Future:

```text
Isaac Sim
        ↓
Policy training
        ↓
SO-101 deployment
```

This is a major robot-learning research direction because simulation can generate more varied data than manual teleoperation alone.

### Direction 6: Language-Conditioned Robot Control

Current instruction:

```text
Pick cube
```

Future instructions:

```text
Pick the red cube
Place the cube in the box
Move the object to the left
```

Relevant VLA models:

```text
SmolVLA
OpenVLA
Pi0
GR00T
```

### Direction 7: Reinforcement Learning Fine-Tuning

Current:

```text
Imitation Learning with ACT
```

Future:

```text
ACT initialization
        ↓
Reinforcement learning fine-tuning
        ↓
Improved performance
```

This connects naturally to Isaac Lab and sim-to-real research, where imitation learning can provide a strong initial policy and reinforcement learning can refine performance.

## 15. Interview Summary

A strong concise answer:

```text
I collected teleoperated demonstrations using the SO-101 leader-follower setup, trained an ACT policy with approximately 18k frames across 20 episodes, and deployed intermediate checkpoints on the real robot using LeRobot rollouts. Future work would include collecting larger datasets, comparing ACT against diffusion and VLA-based policies, improving robustness through diverse environments, and exploring sim-to-real transfer using Isaac Sim and reinforcement learning fine-tuning.
```

This connects the project directly to modern robot-learning research:

- Imitation learning.
- Real-robot deployment.
- Dataset quality and policy evaluation.
- ACT and diffusion-style policy comparison.
- Vision-language-action policies.
- Sim-to-real transfer.
- Reinforcement learning fine-tuning.

## References

- Hugging Face LeRobot project: [https://huggingface.co/lerobot](https://huggingface.co/lerobot)
- Original LeRobot source repository: [https://github.com/huggingface/lerobot](https://github.com/huggingface/lerobot)
- LeRobot documentation: [https://huggingface.co/docs/lerobot/index](https://huggingface.co/docs/lerobot/index)
- User-facing agent guide: [AGENT_GUIDE.md](AGENT_GUIDE.md)
- SO-101 docs: [docs/source/so101.mdx](docs/source/so101.mdx)
- ACT docs: [docs/source/act.mdx](docs/source/act.mdx)
- ACT paper note: [docs/source/policy_act_README.md](docs/source/policy_act_README.md)
- LeRobot source entry points: [pyproject.toml](pyproject.toml)

This repository is based on Hugging Face LeRobot, licensed under Apache License 2.0. It is maintained here for study, code review, and experimentation with robot learning, teleoperation, datasets, ACT training, and real-robot rollout.
