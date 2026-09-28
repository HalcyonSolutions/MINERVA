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

The repository provides preprocessing wrappers for the datasets used by the navigation experiments. They copy the navigation-ready source files into <code>datasets/nlq/&lt;dataset&gt;/</code>, build the graph representation, and create entity/relation vocabularies.

### Kinship

Download the dataset into the location expected by the preprocessing wrapper:

~~~bash
huggingface-cli download HalcyonSolutions/Kinship \
  --repo-type dataset \
  --local-dir ./raw_data/kinship_hinton
~~~

Then preprocess it:

~~~bash
bash scripts/preprocessing/kinship.sh
~~~

The resulting QA file is <code>datasets/nlq/kinship/kinship_qa_nhop.csv</code>.

### MQuAKE-ST

~~~bash
huggingface-cli download HalcyonSolutions/MQuAKE-ST \
  --repo-type dataset \
  --local-dir ./raw_data/mquake_st_dataset

bash scripts/preprocessing/mquake_st.sh
~~~

This creates both the single-answer and multi-answer QA files used by the supplied configurations:

- <code>mquake_sa_qa_nhop.csv</code>
- <code>mquake_ma_qa_nhop.csv</code>

### MetaQA

The preprocessing wrapper expects the navigation-ready MetaQA files under:

~~~text
./raw_data/metaqa_dataset/
├── kg/triplets.txt
├── metadata/node_data.csv
├── metadata/relation_data.csv
├── qa/metaqa_nhop.csv
├── README.md
└── LICENSE
~~~

The dataset source currently referenced by the wrapper is:

https://storage.googleapis.com/halcyon_data/multihop_ds/datasets/MetaQA/index.html

After placing the files in that directory, run:

~~~bash
bash scripts/preprocessing/metaqa.sh
~~~

### What preprocessing creates

The common wrapper <code>scripts/preprocessing/dataset.sh</code> invokes the graph and vocabulary utilities in <code>code/data/preprocessing_scripts/</code>. For the supplied datasets it creates a full graph and the vocabularies required by the agent.

See [Data format](data_format.md) for the generated layout and for instructions on adding a custom dataset.

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

## 6. Important configuration groups

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

## 7. Running a folder of configurations

<code>scripts/bulk_nlq.sh</code> launches every YAML in a directory with a configurable maximum number of concurrent jobs:

~~~bash
bash scripts/bulk_nlq.sh path/to/config_folder 4
~~~

Use a directory containing only training configurations that you actually want to launch; dataset directories in <code>configs/</code> may contain both training and evaluation YAMLs.

## 8. Where to look next

- [Architecture](architecture.md): model and code organization.
- [Data format](data_format.md): graph and QA schemas.
- [Evaluation metrics](metrics.md): exact evaluator behavior and metric definitions.
