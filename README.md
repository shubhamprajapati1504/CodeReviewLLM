# LLM Code Review & Refinement

A personal experiment exploring how large language models can help with two everyday code-review tasks:

- generating useful review comments from a code change
- suggesting a refined code change from a review comment

The project compares two directions: fine-tuning Llama 3 8B with QLoRA and prompting GPT-3.5 with retrieved examples plus static code metadata.

## What I tried

### QLoRA fine-tuning

I converted the code-review data into an instruction-following format and fine-tuned Llama 3 8B with a parameter-efficient QLoRA setup. This keeps the training workflow practical for a machine with 16 GB of VRAM.

The fine-tuned model receives either a diff hunk or a review comment with the original change, then produces a review comment or a proposed refinement.

### Few-shot prompting

I also tested GPT-3.5 in a few-shot setting. For each example, the prompt can include the changed code, a function call graph, a code summary, and relevant demonstrations selected with BM25 retrieval.

## Results

These are results from a 5,000-example test subset used for quick iteration.

### Review-comment generation

| Approach | BLEU-4 | BERTScore |
| --- | ---: | ---: |
| CodeReviewer baseline | 4.28 | 0.8348 |
| Llama 3 8B, QLoRA fine-tuned | 5.27 | 0.8476 |
| GPT-3.5, prompted with call graph + summary | 8.27 | 0.8515 |

### Code-refinement generation

| Approach | BLEU-4 | Exact match | BERTScore |
| --- | ---: | ---: | ---: |
| CodeReviewer baseline | 83.61 | 0.308 | 0.9776 |
| Llama 3 8B, QLoRA fine-tuned | 80.47 | 0.237 | 0.9745 |
| GPT-3.5, prompted with call graph + summary | 79.46 | 0.107 | 0.9704 |

The prompting setup performed best for review comments, while the refinement task remained harder and left room for further iteration.

## Repository layout

```text
Fine-tuning/  Data-preparation and Llama 3 training notebooks
Metric/       BLEU, exact-match, and BERTScore evaluation scripts
Prompting/    Few-shot prompting experiment script and runner
Refinement/   Refinement test data and generated outputs
Review/       Review-comment test data and generated outputs
```

## Getting started

### Fine-tuning

Open the notebooks in `Fine-tuning/` to follow the data-preparation, training, and inference workflow. The training experiment uses the Unsloth and Hugging Face ecosystem for QLoRA fine-tuning.

### Prompting experiments

Run the experiment script from the `Prompting` directory. Add your API key and update the input file names or experiment options as needed.

```bash
python prompt_experiment_script.py \
  --open_key <your-api-key> \
  --model instruct \
  --mode BM25 \
  --number_of_fewshot_sample 5 \
  --length 5000 \
  --type Summary \
  --train_file ref-train-merged.jsonl \
  --test_file ref-test-5000-merged.jsonl
```

`run_experiment.sh` contains the same command as a reusable starting point.

### Evaluation

Install the evaluation dependencies, including `nltk` and `bert-score`, then run the evaluator from the relevant output directory. Update the file names in `Metric/evaluate.py` if you want to score a different prediction file.

```bash
python evaluate.py
```

## Notes

- Training data is not included because of its size; the repository contains the test data and example outputs used for evaluation.
- The dataset is based on Microsoft CodeReviewer. See the [CodeReviewer paper](https://arxiv.org/pdf/2203.09095) for background on the original data and baseline.
- This is an exploratory personal project, so the scripts are intended as a clear starting point rather than a production-ready pipeline.
