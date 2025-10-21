## Other IL Policies

This tutorial provides instruction to train various classic imitation learning policies on the 2025 BEHAVIOR-1K Challenge dataset. 

### Repo Clone

```
git clone https://github.com/StanfordVL/b1k-baselines.git --recurse-submodules
git clone https://github.com/StanfordVL/BEHAVIOR-1K.git
```
This il_lib repo is inspired by Behavior Robot Suite [repo](https://github.com/behavior-robot-suite/brs-algo).

### Installation

IL_LIB is compatible with the BEHAIVOR-1K repo, so you can directly install 

```
conda activate behavior
cd baselines/il_lib
pip install -e .
```

Alternatively, feel free to have it installed in a new conda env, although you need to have the complete behavior stack installed within this conda env to perform online evaluation during training.


### Model Training

We provide a trained WB-VIMA checkpoint [here](https://drive.google.com/file/d/1YTB0XCh32EHyq2svcsYN6HNidk7-8hkJ/view?usp=sharing) for the turning_on_radio task. If you would like to run eval only feel free to skip to the last section. 

Note: in order to run WB-VIMA training, you will need to have point cloud generated locally, we provide a sample point cloud generated for `turning_on_radio` [here](), for more information, checkout the `generate_custon_data.ipynb` within the b1k-baselines repo. 

IL_LIB includes implementations for common behavior cloning baselines, including RGB(D) [Diffusion Poilcy](https://diffusion-policy.cs.columbia.edu/), [3D Diffusion Policy](https://3d-diffusion-policy.github.io/), [ACT](https://tonyzhaozh.github.io/aloha/), [BC-RNN](https://robomimic.github.io/), [WB-VIMA](https://behavior-robot-suite.github.io/). You can find all the config files under `il_lib/configs/arch`. We will use `WB-VIMA` as the example for the following tutorial.

Before running actual training, try perform a fast_dev_run to make sure everything is ok:

```
python train.py data_dir=$DATA_PATH robot=r1pro task=behavior task.name=turning_on_radio arch=wbvima trainer.fast_dev_run=true +eval=behavior headless=false
```

If the train & eval sanity check passes and you see OmniGibson starts online evaluation, you can safely exit the program, the run the following command to launch the actual training:

```
python train.py data_dir=$DATA_PATH robot=r1pro task=behavior task.name=turning_on_radio arch=wbvima
```

Overwrite any parameters in the CLI if needed.


### Evaluation

After finetuning, you can run evaluation by following the steps below:

1. Deploy finetuned checkpoint:

    ```
    python serve.py robot=r1pro task=behavior arch=wbvima ckpt_path=$CKPT_ATH
    ```
    This opens a connection listening on 0.0.0.0:8000.


2. Run the evaluation on BEHAVIOR

    Put `wbvima_wrapper.py` under `OmniGibson/omnigibson/learning/wrappers`, then run:
    
    ```
    conda activate behavior 
    python OmniGibson/omnigibson/learning/eval.py policy=websocket task.name=turning_on_radio env_wrapper._target_=omnigibson.learning.wrappers.wbvima_wrapper.WBVIMAWrapper log_path=$LOG_PATH
    ```