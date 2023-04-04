# Pythia: Interpreting Autoregressive Transformers Across Time and Scale

This repository is for EleutherAI's work-in-progress project *Pythia* which combines interpretability analysis and scaling laws to understand how knowledge develops and evolves during training in autoregressive transformers.

## Models

| Params              | n_layers | d_model | n_heads | d_head | Batch Size | Learning Rate | Checkpoints                                                | Evaluations     |
| ------------------- | -------- | ------- | ------- | ------ | ---------- | ------------- | ---------------------------------------------------------- | --------------- |
| Pythia-70M          | 6        | 512     | 8       | 64     | 2M         | 1e-3          | [Here](https://huggingface.co/EleutherAI/pythia-70m)          | Ready           |
| Pythia-70M-Deduped  | 6        | 512     | 8       | 64     | 2M         | 1e-3          | [Here](https://huggingface.co/EleutherAI/pythia-70m-deduped)  | Ready           |
| Pythia-160M         | 12       | 768     | 12      | 64     | 4M         | 6e-4          | [Here](https://huggingface.co/EleutherAI/pythia-160m)         | Ready           |
| Pythia-160M-Deduped | 12       | 768     | 12      | 64     | 4M         | 6e-4          | [Here](https://huggingface.co/EleutherAI/pythia-160m-deduped) | Ready           |
| Pythia-410M         | 24       | 1024    | 16      | 64     | 4M         | 3e-4          | [Here](https://huggingface.co/EleutherAI/pythia-410m)         | Ready           |
| Pythia-410M-Deduped | 24       | 1024    | 16      | 64     | 4M         | 3e-4          | [Here](https://huggingface.co/EleutherAI/pythia-410m-deduped) | Ready           |
| Pythia-1B         | 16       | 2048    | 8       | 256   | 2M         | 3e-4          | [Here](https://huggingface.co/EleutherAI/pythia-1b)         | Ready           |
| Pythia-1B-Deduped | 16       | 2048    | 8       | 256    | 2M         | 3e-4          | [Here](https://huggingface.co/EleutherAI/pythia-1b-deduped) | Ready           |
| Pythia-1.4B         | 24       | 2048    | 16      | 128    | 4M         | 2e-4          | [Here](https://huggingface.co/EleutherAI/pythia-1.4b)         | Ready           |
| Pythia-1.4B-Deduped | 24       | 2048    | 16      | 128    | 4M         | 2e-4          | [Here](https://huggingface.co/EleutherAI/pythia-1.4b-deduped) | Ready           |
| Pythia-2.8B         | 32       | 2560    | 32      | 80     | 2M         | 1.6e-4        | [Here](https://huggingface.co/EleutherAI/pythia-2.8b)         | Ready           |
| Pythia-2.8B-Deduped | 32       | 2560    | 32      | 80     | 2M         | 1.6e-4        | [Here](https://huggingface.co/EleutherAI/pythia-2.8b-deduped) | Ready           |
| Pythia-6.9B         | 32       | 4096    | 32      | 128    | 2M         | 1.2e-4        | [Here](https://huggingface.co/EleutherAI/pythia-6.9b)         | Ready           |
| Pythia-6.9B-Deduped | 32       | 4096    | 32      | 128    | 2M         | 1.2e-4        | [Here](https://huggingface.co/EleutherAI/pythia-6.9b-deduped) | Ready           |
| Pythia-12B          | 36       | 5120    | 40      | 128    | 2M         | 1.2e-4        | [Here](https://huggingface.co/EleutherAI/pythia-12b)          | Ready |
| Pythia-12B-Deduped  | 36       | 5120    | 40      | 128    | 2M         | 1.2e-4        | [Here](https://huggingface.co/EleutherAI/pythia-12b-deduped)  | Ready |

We train and release a suite of 8 model sizes on 2 different datasets: [the Pile](https://pile.eleuther.ai/), as well as the Pile with deduplication applied.

All 8 model sizes are trained on the exact same data, in the exact same order. Each model saw 299,892,736,000 ~= 299.9B tokens during training, and *143 checkpoints* for each model are saved every 2,097,152,000 ~= 2B tokens, evenly spaced throughout training. This corresponds to just under 1 epoch on the Pile for non-"deduped" models, and ~= 1.5 epochs on the deduped Pile (which contains 207B tokens in 1 epoch).

Config files used to train these models within the [GPT-NeoX library](https://github.com/EleutherAI/gpt-neox) can be found at the `models/` directory within this repository.

We also upload the pre-tokenized data files and a script to reconstruct the dataloader as seen during training for all models. See **Reproducing Training** section for more details.

## Quickstart

All Pythia models are hosted on [the Huggingface hub](https://huggingface.co/EleutherAI). They can be loaded and used via the following code (shown for the 3rd `pythia-70M-deduped` model checkpoint):

```python
from transformers import GPTNeoXForCausalLM, AutoTokenizer

model = GPTNeoXForCausalLM.from_pretrained(
  "EleutherAI/pythia-70m-deduped",
  revision="step3000",
  cache_dir="./pythia-70m-deduped/step3000",
)

tokenizer = AutoTokenizer.from_pretrained(
  "EleutherAI/pythia-70m-deduped",
  revision="step3000",
  cache_dir="./pythia-70m-deduped/step3000",
)

inputs = tokenizer("Hello, I am", return_tensors="pt")
tokens = model.generate(**inputs)
tokenizer.decode(tokens[0])
```

All models were trained for the equivalent of 143000 steps at a batch size of 2,097,152 tokens. Revision/branch `step143000` (e.g. [https://huggingface.co/EleutherAI/pythia-19m-deduped/tree/step143000](https://huggingface.co/EleutherAI/pythia-19m-deduped/tree/step143000)) corresponds exactly to the model checkpoint on the `main` branch of each model.

Models with a batch size of 4M tokens listed were originally trained for 71500 steps instead, and checkpointed every 500 steps. The checkpoints on Huggingface are renamed for consistency with all 2M batch models, so `step1000` is the first checkpoint for the 1.3B that was saved (corresponding to step 500 in training), and `step1000` is likewise the first 6.9B checkpoint that was saved (corresponding to 1000 "actual" steps.)

We additionally have all model checkpoints in the format accepted by the [GPT-NeoX library](https://github.com/EleutherAI/gpt-neox), but do not serve them at scale due to size of optimizer states and anticipated lower demand. If you would like to perform analysis using the models within the GPT-NeoX codebase, or would like the optimizer states, please email hailey@eleuther.ai and stella@eleuther.ai to arrange access.

## Reproducing Training

We provide the training data for replication of our training runs. The [GPT-NeoX library](https://github.com/EleutherAI/gpt-neox) requires the pre-tokenized training data in the form of 2 memory-mapped numpy arrays: a `.bin` and `.idx` file.

We provide these files, hosted on the Hugging Face hub.

To download and use the deduplicated Pile training data, run:

```bash
git lfs clone https://huggingface.co/datasets/EleutherAI/pythia_pile_idxmaps

python utils/unshard_memmap.py --input_file ./pythia_pile_idxmaps/pile_0.87_deduped_text_document-00000-of-00082.bin --num_shards 83 --output_dir ./pythia_pile_idxmaps/
```
This will take over a day to run, though it should not require more than 5 GB of RAM. We recommend downloading this rather than retokenizing the Pile from scratch, in order to preserve the data order seen by the Pythia models.

TODO: forthcoming: more information on how to replicate + relaunch the Pythia training runs, once the data is actually downloaded.


### Dataset Viewer

We provide a tool to view particular portions of the training dataloader used by all models during training, at `utils/batch_viewer.py`.

To run, first substitute the filepath to the downloaded `.bin` and `.idx` files for either the Pile or deduplicated Pile in `utils/dummy_config.yml`.

```python
PYTHONPATH=utils/gpt-neox/ python utils/batch_viewer.py \
  --start_iteration 0 \
  --end_iteration 1000 \
  --mode save \
  --conf_dir utils/dummy_config.yml 
```

Passing `--mode save` will save a separate file containing each batch as a numpy array. 

Passing `--mode custom` will save a dictionary for each batch to a JSONL file--it can be used to compute arbitrary statistics over each batch seen during training.


## Benchmark Scores

We also provide benchmark 0-shot and 5-shot results on a variety of NLP datasets:

- Lambada (`lambada_openai`)
- Wikitext (`wikitext`)
- PiQA (`piqa`)
- SciQ (`sciq`)
- WSC (`wsc`)
- Winogrande (`winogrande`)
- ARC-challenge (`arc_challenge`)
- ARC-easy (`arc_easy`)
- LogiQA (`logiqa`)
- BLiMP (`blimp_*`)
- MMLU (`hendrycksTest*`)

Evaluations were performed in GPT-NeoX using the [LM Evaluation Harness](https://github.com/EleutherAI/lm-evaluation-harness), and are viewable by model and step at `results/json/*` in this repository.

### Plotting Results

We will also provide utilities for creating plots based on the dumped zero and few-shot results. Sample notebook and data format forthcoming.
