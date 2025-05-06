# Trajectory Control Benchmark: HPTR Evaluation Kit

This repository provides an offline benchmarking kit for evaluating trajectory prediction and control methods using HPTR-formatted data. It supports pretrained models such as **Wayformer** and **TrafficSim**, along with classic controllers like **PID** and **LQC**, in a reproducible Docker-based CARLA + ROS2 simulation environment.

---

## Features

- **Plug-and-play predictors**: Wayformer (Google Research) & TrafficSim (Meta AI)
- **Controller comparison**: PID & LQC
- **HPTR-compatible config**: 6 predefined YAMLs with one-line switches
- **Metrics automation**: ADE, FDE, Collision Rate
- **Dockerized environment**: Based on CARLA 0.9.15 + ROS2 Foxy + PyTorch 2.2 + CUDA 12.3
- **Dry-run samples**: Built-in scenes from WOMD→HPTR pipeline

---

## Directory Overview

```
.
├── configs/                  # group1~6 config YAMLs
├── data/                     # dry-run HPTR scenes (auto-generated)
├── models/                   # pretrained weights (not included in repo)
├── results/                  # inference results
├── scripts/
│   └── make_sample_scene.py  # randomly pick 2 scenes from HPTR
├── metrics.py                # ADE/FDE/COL scoring script
├── metrics.json              # sample output JSON
├── requirements.txt          # Python dependencies
├── Dockerfile                # base ROS2+PyTorch image
├── docker-compose.yaml       # CARLA + ROS2 orchestration
├── devcontainer.json         # VSCode DevContainer config
└── README.md
```

---

## Start run

### 1. Build & Run Environment

```bash
docker compose up --build
```

Ensure you have **NVIDIA Container Toolkit** and a GPU-ready system.

### 2. Generate Sample Scenes

```bash
python scripts/make_sample_scene.py
```

### 3. Run Dry-Run Evaluation

```bash
python metrics.py --pred dummy_pred.npy --gt gt.npy --out metrics.json
```

---

## Config Overview (`configs/*.yaml`)

Each YAML config allows toggling:

```yaml
predictor:
  name: "wayformer"        # none | wayformer | trafficsim
  ckpt_path: "models/wayformer.pt"

controller:
  type: "pid"              # pid | lqc
  params:
    kp: 0.8
    ki: 0.1
    kd: 0.2

scenario_list: "lists/scenes.txt"
output_dir: "results/g1"
```

---

## Evaluation Metrics

Implemented in `metrics.py`:

| Metric | Description                                |
|--------|--------------------------------------------|
| ADE    | Average Displacement Error (lower = better)|
| FDE    | Final Displacement Error (last timestep)   |
| COL    | Collision Rate (threshold < 2.0m)          |

Command:

```bash
python metrics.py --pred <pred.npy> --gt <gt.npy> --out <metrics.json>
```
---

## Dependencies

Install Python packages (if outside Docker):

```bash
pip install -r requirements.txt
```

Includes: `numpy`, `scipy`, `pyyaml`, `opencv-python-headless`, `matplotlib`, `wandb`, `rich`...

---

## Notes for Offline Usage

This kit is optimized for lab/cluster environments with:
- **Pre-downloaded models** in `models/`
- **Zero network dependency** during experiments
- **Containerized execution** (`docker run` returns code 0 after smoke test)

---

## Acknowledgements

- [HPTR](https://zhejz.github.io/hptr)
- [TrafficSim (Meta AI)](https://github.com/facebookresearch/traffic-gen)
- [CARLA Simulator](https://carla.org/)
- [WOMD → HPTR](https://huggingface.co/datasets/utias-driving/driving-dojo)
