# SPQR model compression


This repository contains code demonstrating SpQR method for LLM compression.

It accompanies the research paper "SpQR: A Sparse-Quantized Representation for
Almost-Lossless LLM Weight Compression" .

# Installation

### Packages

Install packages from `requirements.txt`:
```bash
pip install -r requirements.txt
```

Note that due to compatibility issues with LLaMA, we recommend using `4.28.dev0` version of `transformers`.

### Downloading model weights and dataset(s)

This scripts assume that model weights are preloaded and stored locally. See `MODEL_PATH` references below.

The scripts can use a variety of datasets for training. To use Red Pajamas, download it locally and then pass the location to the scripts. See `PAJAMAS_PATH` references below.

### Loading / caching datasets and tokenizer

The script will require downloading and caching locally the relevant LLaMA tokenizer and one or few datasets for testing. They will be saved in default locations.

#### Data

The tokenized and preproccessed subset of [RedPajamas](https://huggingface.co/datasets/togethercomputer/RedPajama-Data-1T-Sample) 
(mixture of datasets used for LLaMA training) is located here: `data/red_pajama_n=1024.pth`. 
Below `PAJAMAS_PATH` denotes the path to this subset.

# Launching

### GPU requirements
This code was developed and tested using a single A100 GPU with 80GB GPU RAM. It may successfully run on GPUs with 32 - 40GB   

### Model downloading
The code requires the LLaMA model to be dowloaded in Huggingface format and saved locally. The scripts below require such model folder path as argument.

### Perplexity benchmarks:
This script compresses the model and then tests its performance in terms of perplexity using Wikitext2, 
C4, and Penn Treebank datasets. Note that the perplexity is related to the loss used in the article as `loss = log2(perplexity)`

The command to launch the script should look like this: 

```
export MODEL_PATH=<INSERT PATH_TO_MODEL_DIR>
export PAJAMAS_PATH=<INSERT PATH TO PAJAMAS DIR>

python main.py $MODEL_PATH custom \
    --load_from_saved=$PAJAMAS_PATH \
    --wbits 4 \
    --groupsize 16 \
    --perchannel \
    --qq_scale_bits 3 \
    --qq_zero_bits 3 \
    --qq_groupsize 16 \
    --fit_quantizer_without_outliers \
    --outlier_threshold=0.2 \
    --permutation_order act_order \
    --percdamp 1e0 \
    --nsamples 128 
```
The command above runs near-lossless compression as described in the article. Adjusting the above parameters allows for tighter compression with a slightly greater loss. 

Note the launch arguments:
- `<PATH_TO_MODEL_DIR>` - path to model folder, which contains `config.json `
- `one of [c4, ptb, wikitext2, custom]` -- name of dataset to use for compression
- `--load_from_saved` - path to preprocessed and tokenized dataset (if `custom` chosen). Otherwise do not specify.
- `--wbits 3` -- number of bits for quantized weights representation
- `--groupsize 16` -- size of first-order groups for compression
- `--qq_groupsize 16` -- size of second-order (quantized) groups for compression
- `--qq_scale_bits 3 --qq_zero_bits 3` -- bit sizes for quantizing first order weights' scale and zeros.
- `--fit_quantizer_without_outliers` -- when finding optimal quantizer params, remove any points that would be declared outliers
run `python main.py --help` for more details on command line arguments, including compression parameters.

### LM Evaluation Harness benchmark.

To perform zero-shot evaluation, we use [lm-eval-harness](https://github.com/EleutherAI/lm-evaluation-harness) framework with slight modifications. The LICENSE and CODEOWNERS files inside lm-evaluation-harness refer to the original authors of lm-eval-harness and not the authors of this paper.

For instructions about zero-shot evaluation refer to `README.md` inside `lm-evaluation-harness` directory.

