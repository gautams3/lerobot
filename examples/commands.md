# Using the [SO-100](https://github.com/TheRobotStudio/SO-ARM100) with LeRobot

## Table of Contents

- [Using the SO-100 with LeRobot](#using-the-so-100-with-lerobot)
  - [Table of Contents](#table-of-contents)
  - [E. Calibrate](#e-calibrate)
      - [a. Manual calibration of follower arm](#a-manual-calibration-of-follower-arm)
      - [b. Manual calibration of leader arm](#b-manual-calibration-of-leader-arm)
  - [F. Teleoperate](#f-teleoperate)
      - [a. Teleop with displaying cameras](#a-teleop-with-displaying-cameras)
  - [G. Record a dataset](#g-record-a-dataset)
  - [H. Visualize a dataset](#h-visualize-a-dataset)
  - [I. Replay an episode](#i-replay-an-episode)
  - [J. Train a policy](#j-train-a-policy)
  - [K. Evaluate your policy](#k-evaluate-your-policy)
  - [L. More Information](#l-more-information)
  - [M. Train and Eval Scripts](#m-train-and-eval-scripts)
    - [Upload models to huggingface](#upload-models-to-huggingface)
    - [ACT](#act)
    - [Diffusion](#diffusion)
    - [pi0](#pi0)


## E. Calibrate

Next, you'll need to calibrate your SO-100 robot to ensure that the leader and follower arms have the same position values when they are in the same physical position. This calibration is essential because it allows a neural network trained on one SO-100 robot to work on another.

#### a. Manual calibration of follower arm

> [!IMPORTANT]
> Contrarily to step 6 of the [assembly video](https://youtu.be/FioA2oeFZ5I?t=724) which illustrates the auto calibration, we will actually do manual calibration of follower for now.

You will need to move the follower arm to these positions sequentially:

| 1. Zero position                                                                                                                                             | 2. Rotated position                                                                                                                                                   | 3. Rest position                                                                                                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <img src="../media/so100/follower_zero.webp?raw=true" alt="SO-100 follower arm zero position" title="SO-100 follower arm zero position" style="width:100%;"> | <img src="../media/so100/follower_rotated.webp?raw=true" alt="SO-100 follower arm rotated position" title="SO-100 follower arm rotated position" style="width:100%;"> | <img src="../media/so100/follower_rest.webp?raw=true" alt="SO-100 follower arm rest position" title="SO-100 follower arm rest position" style="width:100%;"> |

Make sure both arms are connected and run this script to launch manual calibration:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=calibrate \
  --control.arms='["main_follower"]'
```

#### b. Manual calibration of leader arm
Follow step 6 of the [assembly video](https://youtu.be/FioA2oeFZ5I?t=724) which illustrates the manual calibration. You will need to move the leader arm to these positions sequentially:

| 1. Zero position                                                                                                                                       | 2. Rotated position                                                                                                                                             | 3. Rest position                                                                                                                                       |
| ------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| <img src="../media/so100/leader_zero.webp?raw=true" alt="SO-100 leader arm zero position" title="SO-100 leader arm zero position" style="width:100%;"> | <img src="../media/so100/leader_rotated.webp?raw=true" alt="SO-100 leader arm rotated position" title="SO-100 leader arm rotated position" style="width:100%;"> | <img src="../media/so100/leader_rest.webp?raw=true" alt="SO-100 leader arm rest position" title="SO-100 leader arm rest position" style="width:100%;"> |

Run this script to launch manual calibration:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=calibrate \
  --control.arms='["main_leader"]'
```

## F. Teleoperate

**Simple teleop**
Then you are ready to teleoperate your robot! Run this simple script (it won't connect and display the cameras):
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --robot.cameras='{}' \
  --control.type=teleoperate \
  --control.teleop_time_s=10
```


#### a. Teleop with displaying cameras
Follow [this guide to setup your cameras](https://github.com/huggingface/lerobot/blob/main/examples/7_get_started_with_real_robot.md#c-add-your-cameras-with-opencvcamera). Then you will be able to display the cameras on your computer while you are teleoperating by running the following code. This is useful to prepare your setup before recording your first dataset.

> **NOTE:** To visualize the data, enable `--control.display_data=true`. This streams the data using `rerun`.

```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100 \
  --control.type=teleoperate \
  --control.display_data=true \
  --control.teleop_time_s=10
```

## G. Record a dataset

Once you're familiar with teleoperation, you can record your first dataset with SO-100.

If you want to use the Hugging Face hub features for uploading your dataset and you haven't previously done it, make sure you've logged in using a write-access token, which can be generated from the [Hugging Face settings](https://huggingface.co/settings/tokens):
```bash
huggingface-cli login --token ${HUGGINGFACE_TOKEN} --add-to-git-credential
```

Store your Hugging Face repository name in a variable to run these commands:
```bash
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
```

Record 2 episodes and upload your dataset to the hub:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100wrist \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=${HF_USER}/so100wrist_test2 \
  --control.tags='["so100wrist","tutorial"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=10 \
  --control.push_to_hub=false \
  --control.display_data=true \
  --control.resume=true
```

Note: You can resume recording by adding `--control.resume=true`.

## H. Visualize a dataset

If you uploaded your dataset to the hub with `--control.push_to_hub=true`, you can [visualize your dataset online](https://huggingface.co/spaces/lerobot/visualize_dataset) by copy pasting your repo id given by:
```bash
echo ${HF_USER}/so100_test
```

If you didn't upload with `--control.push_to_hub=false`, you can also visualize it locally with (a window can be opened in the browser `http://127.0.0.1:9090` with the visualization tool):
```bash
python lerobot/scripts/visualize_dataset_html.py \
  --repo-id ${HF_USER}/so100_test \
  --local-files-only 1
```

## I. Replay an episode

Now try to replay the first episode on your robot:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100wrist \
  --control.type=replay \
  --control.fps=30 \
  --control.repo_id=${HF_USER}/so100wrist_test2 \
  --control.episode=0
```

## J. Train a policy

To train a policy to control your robot, use the [`python lerobot/scripts/train.py`](../lerobot/scripts/train.py) script. A few arguments are required. Here is an example command:
```bash
python lerobot/scripts/train.py \
  --dataset.repo_id=${HF_USER}/so100_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so100_test \
  --job_name=act_so100_test \
  --policy.device=cuda \
  --wandb.enable=true
```

Let's explain it:
1. We provided the dataset as argument with `--dataset.repo_id=${HF_USER}/so100_test`.
2. We provided the policy with `policy.type=act`. This loads configurations from [`configuration_act.py`](../lerobot/common/policies/act/configuration_act.py). Importantly, this policy will automatically adapt to the number of motor sates, motor actions and cameras of your robot (e.g. `laptop` and `phone`) which have been saved in your dataset.
4. We provided `policy.device=cuda` since we are training on a Nvidia GPU, but you could use `policy.device=mps` to train on Apple silicon.
5. We provided `wandb.enable=true` to use [Weights and Biases](https://docs.wandb.ai/quickstart) for visualizing training plots. This is optional but if you use it, make sure you are logged in by running `wandb login`.

Training should take several hours. You will find checkpoints in `outputs/train/act_so100_test/checkpoints`.

To resume training from a checkpoint, below is an example command to resume from `last` checkpoint of the `act_so100_test` policy:
```bash
python lerobot/scripts/train.py \
  --config_path=outputs/train/act_so100_test/checkpoints/last/pretrained_model/train_config.json \
  --resume=true
```

## K. Evaluate your policy

You can use the `record` function from [`lerobot/scripts/control_robot.py`](../lerobot/scripts/control_robot.py) but with a policy checkpoint as input. For instance, run this command to record 10 evaluation episodes:
```bash
python lerobot/scripts/control_robot.py \
  --robot.type=so100wrist \
  --control.type=record \
  --control.fps=30 \
  --control.single_task="Grasp a lego block and put it in the bin." \
  --control.repo_id=${HF_USER}/eval_diffusion_so100wrist_test1 \
  --control.tags='["tutorial", "so100wrist"]' \
  --control.warmup_time_s=5 \
  --control.episode_time_s=30 \
  --control.reset_time_s=30 \
  --control.num_episodes=10 \
  --control.display_data=true \
  --control.push_to_hub=true \
  --control.policy.path=outputs/train/diffusion_so100wrist_test
```

As you can see, it's almost the same command as previously used to record your training dataset. Two things changed:
1. There is an additional `--control.policy.path` argument which indicates the path to your policy checkpoint with  (e.g. `outputs/train/eval_act_so100_test/checkpoints/last/pretrained_model`). You can also use the model repository if you uploaded a model checkpoint to the hub (e.g. `${HF_USER}/act_so100_test`).
2. The name of dataset begins by `eval` to reflect that you are running inference (e.g. `${HF_USER}/eval_act_so100_test`).

## L. More Information

Follow this [previous tutorial](https://github.com/huggingface/lerobot/blob/main/examples/7_get_started_with_real_robot.md#4-train-a-policy-on-your-data) for a more in-depth tutorial on controlling real robots with LeRobot.

> [!TIP]
>  If you have any questions or need help, please reach out on [Discord](https://discord.com/invite/s3KuuzsPFb) in the channel [`#so100-arm`](https://discord.com/channels/1216765309076115607/1237741463832363039).

## M. Train and Eval Scripts
### Upload models to huggingface
CoPilot: If you want to upload your models to the Hugging Face hub, you can use the `push_to_hub` argument in the training and evaluation scripts.
Let MYMODEL be the name of your model.
1. Create model on huggingface.co with name MYMODEL
2. `cd /path/to/your/model`
3. `huggingface-cli upload salhotra/MYMODEL .`

### ACT
1. Train
```bash
TODO
```

1. Eval

```bash
# Make sure you have the correct environment variables set
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
# Run the evaluation script with the correct parameters
python lerobot/scripts/control_robot.py   --robot.type=so100wrist   --control.type=record   --control.fps=30   --control.single_task="Grasp a lego block and put it in the bin."   --control.repo_id=${HF_USER}/eval_act_so100wrist_test2   --control.tags='["tutorial", "so100wrist"]'   --control.warmup_time_s=5   --control.episode_time_s=30   --control.reset_time_s=30   --control.num_episodes=3   --control.display_data=true   --control.push_to_hub=true   --control.policy.path=outputs/train/3val27train/checkpoints/100000/pretrained_model --control.resume=true
```

### Diffusion

### pi0
1. Train
```bash
python lerobot/scripts/train.py --dataset.repo_id=salhotra/so100wrist_test2 --policy.path=lerobot/pi0 --output_dir=outputs/train/lerobot_pi0_so100wrist_test --job_name=lerobot_pi0_so100wrist_test --policy.device=cuda --wandb.enable=true
```

1. Eval
```bash
# Make sure you have the correct environment variables set
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
# Run the evaluation script with the correct parameters
python lerobot/scripts/control_robot.py   --robot.type=so100wrist   --control.type=record   --control.fps=30   --control.single_task="Grasp a lego block and put it in the bin."   --control.repo_id=${HF_USER}/eval_lerobot_pi0_so100wrist_test   --control.tags='["tutorial", "so100wrist"]'   --control.warmup_time_s=5   --control.episode_time_s=30   --control.reset_time_s=30   --control.num_episodes=3   --control.display_data=true   --control.push_to_hub=true   --control.policy.path=outputs/train/lerobot_pi0_so100wrist_test/checkpoints/100000/pretrained_model --control.resume=true
```
