## Pi0

This toturial provides a simplest version instruction to finetune Pi0 on the 2025 BEHAVIOR-1K Challenge dataset. 

### Repo Clone

```
git clone https://github.com/StanfordVL/b1k-baselines.git --recurse-submodules
git clone https://github.com/StanfordVL/BEHAVIOR-1K.git
```
This finetuning instruction is adapted from the original [openpi repo](https://github.com/Physical-Intelligence/openpi).


### Installation

Openpi use [uv](https://docs.astral.sh/uv/) to manage Python dependencies. See the [uv installation instructions](https://docs.astral.sh/uv/getting-started/installation/) to set it up. Once uv is installed, run the following to set up the environment:

```
cd baselines/openpi
GIT_LFS_SKIP_SMUDGE=1 uv sync
GIT_LFS_SKIP_SMUDGE=1 uv pip install -e .

source .venv/bin/activate

# Install behavior for server deploy 
cd $PATH_TO_BEHAVIOR_1K
uv pip install -e bddl
uv pip install -e OmniGibson[eval]
```

### Finetune OpenPi

We provide a Pi0 checkpoint for:
    
- turning_on_radio task [here](https://drive.google.com/file/d/1YU7evHBj7vfjmE-tNK-Rbie8ytholQTc/view?usp=sharing). This checkpoint has been trained for 50k steps.

- picking_up_trash task [here](https://drive.google.com/file/d/1G_ACu3uUP_9RmXDgqa7307aFt28G-vJN/view?usp=sharing). This checkpoint has been trained for 50k steps.

If you would like to run eval only feel free to skip to the last section. 

Before we can run training, we need to compute the normalization statistics for the training data. Change line 98 of `compute_norm_stats.py` to be the task name you want (or None to include all tasks), then run the script below

```
uv run scripts/compute_norm_stats.py --config-name pi0_b1k
```
This will create `norm_stats.json` under `assets/pi0_b1k/behavior-1k/2025-challenge-demos`


After this, change line 137 of `data_loader.py` to be the task name you want (or None to include all tasks), then run the following command to fintune OpenPi:

```
XLA_PYTHON_CLIENT_MEM_FRACTION=0.9 uv run scripts/train_val.py pi0_b1k \
    --exp_name="openpi_$(date +%Y%m%d_%H%M%S)" \
    --overwrite \
    --batch_size=64 \
    --num_train_steps=50000 \
    --weight_loader.params_path=gs://openpi-assets/checkpoints/pi0_base/params
```


### Evaluation

After finetuning, you can run evaluation by following the steps below:

1. Deploy finetuned checkpoint:

    ```
    source .venv/bin/activate
    uv run scripts/serve_b1k.py --task_name=$TASK_NAME policy:checkpoint --policy.config=pi0_b1k --policy.dir=$PATH_TO_CKPT
    ```
    This opens a connection listening on 0.0.0.0:8000.


2. Run the evaluation on BEHAVIOR:

    Assume you have behavior env installed (check https://github.com/StanfordVL/BEHAVIOR-1K for more details), run the following command within the BEHAVIOR-1K directory:
    ```
    conda activate behavior 
    python OmniGibson/omnigibson/learning/eval.py policy=websocket task.name=turning_on_radio log_path=$LOG_PATH
    ```
