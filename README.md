# MINERVA: Natural-Language KG Navigation

**Official MINERVA adaptation for [THESEUS](https://github.com/HalcyonSolutions/THESEUS)**

This repository adapts the reinforcement-learning navigation framework from [MINERVA](https://arxiv.org/abs/1711.05851) to **natural-language multi-hop knowledge graph question answering**. Instead of conditioning the agent on a symbolic query of the form (h, r, ?), the agent receives a **natural-language question**, a **topic entity**, and a **knowledge graph**, then navigates an explicit sequence of graph edges toward an answer.

![MINERVA KG Navigation](images/minerva_navigation.gif)
*At each hop, MINERVA uses the natural-language question together with its recurrent path state to score the executable relation–entity actions available from the current node. The selected edge moves the agent to the next entity, producing an explicit reasoning path whose terminal entity is used as the predicted answer.*

This implementation is used in [*Theseus in the Graph: Towards Traceable Multi-Hop Graph Navigation*](https://arxiv.org/abs/2609.14528).

> **Project hub:** [THESEUS](https://github.com/HalcyonSolutions/THESEUS) — datasets, other adapted navigation agents, pretrained checkpoints, and shared evaluation resources.  
> **Original MINERVA paper:** [Go for a Walk and Arrive at the Answer](https://arxiv.org/abs/1711.05851)  
> **Symbolic-query / KGC branch:** [minerva_tf1](https://github.com/HernandezEduin/MINERVA/tree/minerva_tf1)

## What is included

- Natural-language question conditioning with transformer embeddings.
- Reinforcement-learning graph navigation with beam-search evaluation.
- Single-answer and multi-answer KGQA.
- Directed and undirected graph settings.
- Optional STOP and RESTART actions.
- Explicit trajectory logging and path-fidelity evaluation.
- Dataset-specific configurations for **Kinship**, **MQuAKE-ST**, and **MetaQA**.
- Pretrained MINERVA checkpoints for three seeds, released through **THESEUS**.
- Structural calibration baselines: **RW-Ans_MC** and the **Shortest Path Oracle**.
- Preprocessing and baseline scripts under <code>scripts/</code>.

## Quick start

Run commands from the repository root.

### 1. Create the environment

~~~bash
conda env create -f environment.yml
conda activate minerva
~~~

Python 3.9 is used by the provided environment. A pip-only installation is also possible with <code>pip install -r requirements.txt</code>.

### 2. Prepare a dataset

Kinship is the smallest bundled example workflow:

~~~bash
huggingface-cli download HalcyonSolutions/Kinship \
  --repo-type dataset \
  --local-dir ./raw_data/kinship_hinton

bash scripts/preprocessing/kinship.sh
~~~

The preprocessing scripts create the graph and vocabularies expected by MINERVA under <code>datasets/nlq/</code>.

### 3. Train

~~~bash
bash scripts/run_nlq.sh configs/kinship/train.yaml 0
~~~

The final argument is an optional GPU ID. Omit it to run on CPU.

### 4. Evaluate

Set <code>model_load_dir</code> in the corresponding evaluation YAML to the checkpoint you want to load, then run:

~~~bash
bash scripts/run_eval.sh configs/kinship/evaluate.yaml 0
~~~

## Current experiment configurations

| Dataset | Training | Evaluation |
| --- | --- | --- |
| Kinship | <code>configs/kinship/train.yaml</code> | <code>configs/kinship/evaluate.yaml</code> |
| MQuAKE-ST single-answer | <code>configs/mquake_st/train_single.yaml</code> | <code>configs/mquake_st/evaluate_single.yaml</code> |
| MQuAKE-ST multi-answer | <code>configs/mquake_st/train_multi.yaml</code> | <code>configs/mquake_st/evaluate_multi.yaml</code> |
| MetaQA | <code>configs/metaqa/train.yaml</code> | <code>configs/metaqa/evaluate.yaml</code> |

## Documentation

| Guide | Contents |
| --- | --- |
| [Getting started](docs/getting_started.md) | Environment setup, datasets, pretrained checkpoints, training, evaluation, and baseline scripts |
| [Data format](docs/data_format.md) | Graph files, QA CSV schema, multi-answer data, paths, and custom datasets |
| [Architecture](docs/architecture.md) | How MINERVA is adapted from symbolic queries to natural-language graph navigation |
| [Evaluation metrics](docs/metrics.md) | Hits@K, MRR, F1_SG, F1_REL, PED, RED, and structural calibration references |

## Repository layout

~~~text
MINERVA/
├── code/                  # Model, environment, data loading, and preprocessing
├── configs/
│   ├── kinship/
│   ├── mquake_st/
│   └── metaqa/
├── docs/                  # User and evaluation documentation
├── images/
├── scripts/
│   ├── baselines/
│   ├── preprocessing/
│   ├── run_nlq.sh
│   └── run_eval.sh
├── environment.yml
└── requirements.txt
~~~

## Citation

If you use the natural-language graph-navigation adaptation in this repository, please cite *Theseus in the Graph*:

~~~bibtex
@article{hernandez2026theseus,
  author  = {Hernandez, Eduin E. and Garcia, Luis F. and Askar, Nurassyl and Diaz, Sergio A. and Rini, Stefano},
  title   = {Theseus in the Graph: Towards Traceable Multi-Hop Graph Navigation},
  journal = {arXiv preprint arXiv:2609.14528},
  year    = {2026}
}
~~~

Please also cite the original MINERVA work:

~~~bibtex
@inproceedings{minerva,
  title     = {Go for a Walk and Arrive at the Answer: Reasoning Over Paths in Knowledge Bases using Reinforcement Learning},
  author    = {Das, Rajarshi and Dhuliawala, Shehzaad and Zaheer, Manzil and Vilnis, Luke and Durugkar, Ishan and Krishnamurthy, Akshay and Smola, Alex and McCallum, Andrew},
  booktitle = {ICLR},
  year      = {2018}
}
~~~

## License

This repository is released under the [Apache License 2.0](LICENSE).
