# Getting Started

This guide covers the supported workflow for the natural-language MINERVA adaptation in this repository. All commands below assume that your current directory is the repository root.

## 1. Environment

The provided Conda environment uses Python 3.9, TensorFlow 2.11.1, CUDA Toolkit 11.2, and cuDNN 8.1.

~~~bash
conda env create -f environment.yml
conda activate minerva
~~~

If you do not need the Conda-managed CUDA stack, you can install the Python dependencies directly:

~~~bash
python -m pip install -r requirements.txt
~~~

The launcher scripts accept an optional GPU index. If no usable GPU index is supplied, they set CUDA_VISIBLE_DEVICES to an empty value and run on CPU.

~~~bash
# GPU 0
bash scripts/run_nlq.sh configs/kinship/train.yaml 0

# CPU
bash scripts/run_nlq.sh configs/kinship/train.yaml
~~~

## 2. Dataset preparation

The repository provides preprocessing wrappers for the datasets used by the navigation experiments. They copy the navigation-ready source files into <code>datasets/nlq/&lt;dataset&gt;/</code>, build the graph representation, and create the entity/relation vocabularies required by MINERVA.

The central [THESEUS project page](https://github.com/HalcyonSolutions/THESEUS) indexes the released datasets, Google Cloud mirrors, adapted agents, and pretrained checkpoints.

### Kinship

**Dataset:** [Hugging Face](https://huggingface.co/datasets/HalcyonSolutions/Kinship) · [Google Cloud mirror](https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/Kinship/index.html)

Using Hugging Face:

~~~bash
huggingface-cli download HalcyonSolutions/Kinship \
  --repo-type dataset \
  --local-dir ./raw_data/kinship_hinton
~~~

Alternatively, copy the dataset archive link from the Google Cloud mirror and download it with `wget`:

~~~bash
wget https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/Kinship/kinship_dataset.zip

unzip kinship_dataset.zip -d ./raw_data/
~~~

The archive already contains the directory expected by the preprocessing script:

~~~text
raw_data/
└── kinship_hinton/
    ├── kg/
    ├── qa/
    ├── README.md
    └── LICENSE
~~~

Then preprocess the dataset:

~~~bash
bash scripts/preprocessing/kinship.sh
~~~

The processed dataset is written under:

~~~text
datasets/nlq/kinship/
~~~

including the QA file:

~~~text
kinship_qa_nhop.csv
~~~

### MQuAKE-ST

**Dataset:** [Hugging Face](https://huggingface.co/datasets/HalcyonSolutions/MQuAKE-ST) · [Google Cloud mirror](https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/MQuAKE_ST/index.html)

Using Hugging Face:

~~~bash
huggingface-cli download HalcyonSolutions/MQuAKE-ST \
  --repo-type dataset \
  --local-dir ./raw_data/mquake_st_dataset
~~~

Alternatively, using the Google Cloud mirror:

~~~bash
wget https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/MQuAKE_ST/mquake_st_dataset.zip

unzip mquake_st_dataset.zip -d ./raw_data/
~~~

The archive already contains the directory expected by the preprocessing script:

~~~text
raw_data/
└── mquake_st_dataset/
    ├── kg/
    ├── metadata/
    ├── qa/
    ├── README.md
    └── LICENSE
~~~

Then preprocess the dataset:

~~~bash
bash scripts/preprocessing/mquake_st.sh
~~~

The processed dataset is written under:

~~~text
datasets/nlq/mquake_st/
~~~

and includes both QA settings used by the provided configurations:

~~~text
mquake_sa_qa_nhop.csv
mquake_ma_qa_nhop.csv
~~~

### MetaQA

**Dataset:** [Original MetaQA repository](https://github.com/yuyuz/MetaQA) · [THESEUS Google Cloud mirror](https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/MetaQA/index.html)

MINERVA uses the navigation-ready THESEUS version rather than the original MetaQA directory layout.

Download the archive from the Google Cloud mirror and extract it under `raw_data/`: 

~~~bash
wget https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/MetaQA/metaqa_dataset.zip

unzip metaqa_dataset.zip -d ./raw_data/
~~~

The archive already contains the directory expected by the preprocessing script:

~~~text
raw_data/
└── metaqa_dataset/
    ├── kg/
    │   └── triplets.txt
    ├── metadata/
    │   ├── node_data.csv
    │   └── relation_data.csv
    ├── qa/
    │   └── metaqa_nhop.csv
    ├── README.md
    └── LICENSE
~~~

Then preprocess the dataset:

~~~bash
bash scripts/preprocessing/metaqa.sh
~~~

The processed dataset is written under:

~~~text
datasets/nlq/metaqa/
~~~

including:

~~~text
metaqa_qa_nhop.csv
~~~

### What preprocessing creates

The dataset wrappers under <code>scripts/preprocessing/</code> copy the required source files and invoke the shared preprocessing utilities in <code>code/data/preprocessing_scripts/</code>.

For the supplied datasets, preprocessing creates the graph representation and entity/relation vocabularies required by MINERVA, resulting in a layout similar to:

~~~text
datasets/nlq/<dataset>/
├── triplets.txt
├── full_graph.txt
├── <qa-file>.csv
├── vocab/
│   ├── entity_vocab.json
│   └── relation_vocab.json
└── ...
~~~

See [Data format](data_format.md) for the complete schema and instructions for adding a custom dataset.

## 3. Dataset-specific configurations

Current public examples are organized by dataset rather than in one shared NLQ configuration directory.

| Dataset | Train config | Evaluation config |
| --- | --- | --- |
| Kinship | <code>configs/kinship/train.yaml</code> | <code>configs/kinship/evaluate.yaml</code> |
| MQuAKE-ST single-answer | <code>configs/mquake_st/train_single.yaml</code> | <code>configs/mquake_st/evaluate_single.yaml</code> |
| MQuAKE-ST multi-answer | <code>configs/mquake_st/train_multi.yaml</code> | <code>configs/mquake_st/evaluate_multi.yaml</code> |
| MetaQA | <code>configs/metaqa/train.yaml</code> | <code>configs/metaqa/evaluate.yaml</code> |

Older experiment YAMLs may remain under <code>configs/nlq/</code>, but the dataset-specific directories above are the maintained entry points documented for public use.

## 4. Training

Use <code>scripts/run_nlq.sh</code> with a training YAML:

~~~bash
# Kinship
bash scripts/run_nlq.sh configs/kinship/train.yaml 0

# MQuAKE-ST, single-answer
bash scripts/run_nlq.sh configs/mquake_st/train_single.yaml 0

# MQuAKE-ST, multi-answer
bash scripts/run_nlq.sh configs/mquake_st/train_multi.yaml 0

# MetaQA
bash scripts/run_nlq.sh configs/metaqa/train.yaml 0
~~~

The launcher calls <code>code/model/trainer.py</code> with the selected YAML.

The supplied training configurations use <code>load_model: False</code>. Checkpoints and logs are written below the configured <code>base_output_dir</code>, with run-specific paths assembled by the option loader.

## 5. Evaluation

Evaluation configurations use <code>load_model: True</code> and contain a <code>model_load_dir</code>. Point that field to the checkpoint you want to evaluate.

~~~bash
# Kinship
bash scripts/run_eval.sh configs/kinship/evaluate.yaml 0

# MQuAKE-ST, single-answer
bash scripts/run_eval.sh configs/mquake_st/evaluate_single.yaml 0

# MQuAKE-ST, multi-answer
bash scripts/run_eval.sh configs/mquake_st/evaluate_multi.yaml 0

# MetaQA
bash scripts/run_eval.sh configs/metaqa/evaluate.yaml 0
~~~

The evaluation launcher calls <code>code/model/evaluation.py</code>. When <code>print_paths: True</code>, human-readable trajectory logs are also written for qualitative inspection.

See [Evaluation metrics](metrics.md) for the exact ranking, path-fidelity, answer-coverage, and diagnostic metrics reported by the evaluator.


## 6. Pretrained checkpoints

Pretrained MINERVA checkpoints for the experiments reported in [*Theseus in the Graph*](https://arxiv.org/abs/2609.14528) are released from the central [THESEUS project page](https://github.com/HalcyonSolutions/THESEUS). Each setting is provided for **three random seeds: 0, 42, and 100**.

| Dataset / setting | Checkpoint page | Expected checkpoint prefix |
| --- | --- | --- |
| Kinship | [Download page](https://storage.googleapis.com/halcyon_data/multihop_ds/conferences/all/minerva/kinshiphinton/index.html) | <code>checkpoints/kinship/qa_nhop_reason_3hop_seed&lt;seed&gt;/model/model.ckpt</code> |
| MQuAKE-ST single-answer | [Download page](https://storage.googleapis.com/halcyon_data/multihop_ds/conferences/all/minerva/mquake_st/single_answers/index.html) | <code>checkpoints/mquake_st/sa_qa_nhop_reason_4hop_seed&lt;seed&gt;/model/model.ckpt</code> |
| MQuAKE-ST multi-answer | [Download page](https://storage.googleapis.com/halcyon_data/multihop_ds/conferences/all/minerva/mquake_st/multi_answers/index.html) | <code>checkpoints/mquake_st/ma_qa_nhop_reason_4hop_seed&lt;seed&gt;/model/model.ckpt</code> |
| MetaQA | [Download page](https://storage.googleapis.com/halcyon_data/multihop_ds/conferences/all/minerva/metaqa/index.html) | <code>checkpoints/metaqa/qa_nhop_reason_3hop_seed&lt;seed&gt;/model/model.ckpt</code> |

### Downloading a checkpoint

Each checkpoint page provides one archive per released seed:

- `seed0.zip`
- `seed42.zip`
- `seed100.zip`

Copy the link for the desired seed from the corresponding checkpoint page, download it with `wget`, and extract it into the dataset checkpoint directory.

For example, to download **MQuAKE-ST multi-answer, seed 0**:

~~~bash
wget https://storage.googleapis.com/halcyon_data/multihop_ds/conferences/all/minerva/mquake_st/multi_answers/seed0.zip

unzip seed0.zip -d ./checkpoints/mquake_st/
~~~

The archive already contains the run-specific directory expected by the evaluation configuration. After extraction, the example above produces:

~~~text
checkpoints/
└── mquake_st/
    └── ma_qa_nhop_reason_4hop_seed0/
        ├── config.json
        ├── LICENSE
        ├── model/
        │   ├── checkpoint
        │   ├── model.ckpt.data-00000-of-00001
        │   ├── model.ckpt.index
        │   └── model.ckpt.meta
        ├── scores.txt
        └── test_beam/
~~~

Use the corresponding destination directory for each dataset:

| Dataset | Extract into |
| --- | --- |
| Kinship | <code>./checkpoints/kinship/</code> |
| MQuAKE-ST single-answer | <code>./checkpoints/mquake_st/</code> |
| MQuAKE-ST multi-answer | <code>./checkpoints/mquake_st/</code> |
| MetaQA | <code>./checkpoints/metaqa/</code> |

The current evaluation configurations use <code>seed: 0</code> by default. Their <code>model_load_dir</code> values interpolate the configured seed and path length, so changing <code>seed</code> automatically changes the checkpoint location expected by the evaluator.

For example:

~~~yaml
seed: 42
path_length: 3
model_load_dir: "checkpoints/kinship/qa_nhop_reason_${path_length}hop_seed${seed}/model/model.ckpt"
~~~

To evaluate another released checkpoint, download the corresponding `seed42.zip` or `seed100.zip` archive and update the <code>seed</code> field in the evaluation YAML.

With the checkpoint in place, run evaluation normally:

~~~bash
bash scripts/run_eval.sh configs/kinship/evaluate.yaml 0
~~~

## 7. Structural calibration baselines

The repository includes the non-learned structural calibration references used in [*Theseus in the Graph*](https://arxiv.org/abs/2609.14528), including the definitions discussed in Appendix A.4. These references operate on the **actual evaluator navigation graph and action space** rather than serving as learned KGQA systems.

- **RW-Ans_MC / unbiased random walk:** samples uniform random navigation trajectories. The supplied scripts use **100 walks per question** and, by default, the three seeds **0, 42, and 100**. The terminal answer-hit rate is the Monte Carlo <code>RW-Ans_MC</code> calibration; the same sampled trajectories are also evaluated with PED, RED, F1_SG, and F1_REL when the required references are available.
- **Shortest Path Oracle:** is given the valid answer set and finds a shortest graph path from the topic entity to a valid answer, but it does **not** use the natural-language question. Its trajectory is evaluated with the same path-fidelity metrics. It is a structural reference, not a path-fidelity upper or lower bound.

Dataset wrappers are provided under <code>scripts/baselines/</code>:

| Setting | Command | Calibration references |
| --- | --- | --- |
| Kinship | <code>bash scripts/baselines/run_kinship.sh</code> | RW-Ans_MC + Shortest Path Oracle |
| MQuAKE-ST single-answer | <code>bash scripts/baselines/run_mquake_st_sa.sh</code> | RW-Ans_MC + Shortest Path Oracle |
| MQuAKE-ST multi-answer | <code>bash scripts/baselines/run_mquake_st_ma.sh</code> | RW-Ans_MC + Shortest Path Oracle |
| MetaQA | <code>bash scripts/baselines/run_metaqa.sh</code> | RW-Ans_MC |

With no seed arguments, the wrappers run <code>0 42 100</code>. To run only selected seeds, pass them explicitly:

~~~bash
bash scripts/baselines/run_kinship.sh 42
bash scripts/baselines/run_mquake_st_sa.sh 0 100
~~~

Machine-readable results are written below <code>output/&lt;dataset&gt;/baselines/</code>. In the random-walk JSON output, the paper's <code>RW-Ans_MC</code> quantity is stored under the summary key <code>RW_Ans</code>.

See [Evaluation metrics](metrics.md#15-structural-calibration-references) for the interpretation of these references and [<code>code/baselines/</code>](https://github.com/HernandezEduin/MINERVA/tree/master/code/baselines) for the implementations.

## 8. Important configuration groups

The YAML files expose the same options as <code>code/options.py</code>. The most commonly changed groups are:

**Data and question input**

- <code>data_input_dir</code>
- <code>raw_QAData_path</code>
- <code>cached_QAMetaData_path</code>
- <code>question_tokenizer_name</code>
- <code>question_format</code>
- <code>evaluate_paraphrases</code>

**Graph navigation**

- <code>use_full_graph</code>
- <code>use_directed_graph</code>
- <code>max_num_actions</code>
- <code>path_length</code>
- <code>use_stop_signal</code>
- <code>use_restart_signal</code>

**Model**

- <code>embedding_size</code>
- <code>hidden_size</code>
- <code>use_entity_embeddings</code>
- <code>train_entity_embeddings</code>
- <code>train_relation_embeddings</code>
- <code>projection_adapter</code>
- <code>projection_layers</code>
- <code>projection_hidden</code>

**Training and decoding**

- <code>num_rollouts</code>
- <code>test_rollouts</code>
- <code>learning_rate</code>
- <code>gamma</code>
- <code>beta</code>
- <code>use_beam</code>
- <code>pool</code>

**Trajectory evaluation**

- <code>path_segment_policy</code>
- <code>print_paths</code>
- <code>print_predictions</code>

The default trajectory policy in <code>code/options.py</code> is <code>final_segment_truncate</code>: evaluation keeps the final attempt after the last RESTART and stops the evaluated path at STOP.

## 9. Running a folder of configurations

<code>scripts/bulk_nlq.sh</code> launches every YAML in a directory with a configurable maximum number of concurrent jobs:

~~~bash
bash scripts/bulk_nlq.sh path/to/config_folder 4
~~~

Use a directory containing only training configurations that you actually want to launch; dataset directories in <code>configs/</code> may contain both training and evaluation YAMLs.

## 10. Where to look next

- [Architecture](architecture.md): model and code organization.
- [Data format](data_format.md): graph and QA schemas.
- [Evaluation metrics](metrics.md): exact evaluator behavior, metric definitions, and structural calibration references.
- [THESEUS](https://github.com/HalcyonSolutions/THESEUS): central project landing page for datasets, adapted agents, and released checkpoints.
