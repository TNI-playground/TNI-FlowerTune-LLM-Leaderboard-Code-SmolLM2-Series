# FlowerTune LLM on Code Dataset

This directory conducts federated instruction tuning with pretrained SmolLM2 Series Models: [SmolLM2-135M-Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-135M-Instruct), [SmolLM2-360M-Instruct](https://huggingface.co/HuggingFaceTB/SmolLM2-360M-Instruct), [SmolLM2-135M](https://huggingface.co/HuggingFaceTB/SmolLM2-135M) and [SmolLM2-360M](https://huggingface.co/HuggingFaceTB/SmolLM2-360M-Instruct) on a [Code dataset](https://huggingface.co/datasets/flwrlabs/code-alpaca-20k).
We use [Flower Datasets](https://flower.dev/docs/datasets/) to download, partition and preprocess the dataset.
Flower's Simulation Engine is used to simulate the LLM fine-tuning process in federated way,
which allows users to perform the training on a single GPU.


## Methodology

This baseline performs federated LLM fine-tuning with [LoRA](https://arxiv.org/pdf/2106.09685) using the [🤗PEFT](https://huggingface.co/docs/peft/en/index) library.
The clients' models are aggregated with FedAvg strategy.
This provides a baseline performance for the leaderboard of Code challenge.


## Environments setup

Project dependencies are defined in `pyproject.toml`. Install them in an activated Python environment with:

```shell
pip install -e .
```

## Experimental setup

The dataset is divided into 10 partitions in an IID fashion, a partition is assigned to each ClientApp.
We randomly sample a fraction (0.2) of the total nodes to participate in each round, for a total of `200` rounds.
All settings are defined in `pyproject.toml`.

> [!IMPORTANT]
> Please note that `[tool.flwr.app.config.static]` and `options.num-supernodes` under `[tool.flwr.federations.local-simulation]` are not allowed to be modified for fair competition if you plan to participated in the [LLM leaderboard](https://flower.ai/benchmarks/llm-leaderboard).


## Running the challenge

Follow the instruction [here](https://huggingface.co/docs/huggingface_hub/en/quick-start#login-command) to log in your account. Note you only need to complete this stage once in your development machine:

```bash
huggingface-cli login
```

Run the challenge with default config values.
The configs are defined in `[tool.flwr.app.config]` entry of `pyproject.toml`, and are loaded automatically.

```bash
flwr run
```

### Benchmark

| Challenges                       | mbpp       |  humaneval |  multiple-js|  multiple-cpp |  Avg       |
| :--------:                       | :--------: | :--------: | :--------:  | :--------:    | :--------: |
|[SmolLM2-135M-Instruct](https://drive.google.com/drive/folders/1H-Ln1lzNcYJf6Vm8L12AIB40x4LANuhe?usp=drive_link) (200Rounds) |   7.20     |   5.48     |    5.59     |    4.96       |  5.80      |
|[SmolLM2-135M](https://drive.google.com/drive/folders/13_NqzdhhjSGaAO9BTh0XDn9bNYQ7_VQP?usp=drive_link) (200Rounds)          |   2.60     |   3.04     |    6.21     |    6.21       |  4.51      |
|[SmolLM2-360M-Instruct](https://drive.google.com/drive/folders/1xOaAqYpyEL72g4W8TjfgE86s3P9uT7cm?usp=drive_link) (200Rounds) |  18.60     |   17.68    |   12.42     |    9.93       |  14.65     |
|[SmolLM2-360M](https://drive.google.com/drive/folders/17HRaRExcMr34BtumpS6gBA8btaVAYHRq?usp=drive_link) (200Rounds)          |  17.20     |   14.02    |   11.80     |    8.69       |  12.92     |

## VRAM consumption

We use models with 4-bit quantization as default. The estimated VRAM consumption per client for each challenge is shown below:
7
|Models|SmolLM2-135M-Instruct (BS=16)|SmolLM2-360M-Instruct (BS=16) |SmolLM2-135M (BS=16)|SmolLM2-360M (BS=16)|
| :----: | :--------:                | :--------:                  | :--------:         | :--------:          |
|VRAM    |     7.54 GB               |         7.20 GB             |      7.17 GB       |      7.28 GB        |
|Comm    |     1417.97 MB            |         2368.03 MB          |     1417.97 MB     |      2377.45 MB     |

You can adjust the CPU/GPU resources you assign to each of the clients based on your device, which are specified with `options.backend.client-resources.num-cpus` and `options.backend.client-resources.num-gpus` under `[tool.flwr.federations.local-simulation]` entry in `pyproject.toml`.


## Model saving

The global PEFT model checkpoints are saved every 5 rounds after aggregation on the sever side as default, which can be specified with `train.save-every-round` under [tool.flwr.app.config] entry in `pyproject.toml`.

> [!NOTE]
> Please provide the last PEFT checkpoint if you plan to participated in the [LLM leaderboard](https://flower.ai/benchmarks/llm-leaderboard).
