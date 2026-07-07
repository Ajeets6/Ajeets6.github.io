# Gurmukhi Small Language Model

Gurmukhi SLM is a research project for bidirectional machine translation between English and Punjabi written in the Gurmukhi script. The project builds the data and tokenizer from scratch, compares sequence-to-sequence and decoder-only Transformer architectures, and refines the decoder-only model through teacher distillation.

## Project Pipeline

1. Combine three English-Punjabi parallel corpora into one consistent schema.
2. Audit and clean the merged corpus.
3. Train a shared 24,000-token bilingual BPE tokenizer.
4. Train a sequence-to-sequence Transformer baseline.
5. Train a modern decoder-only Transformer.
6. Distil translation behavior from a stronger teacher into the decoder-only student.
7. Evaluate the baseline, teacher, and distilled student with automatic and manual translation checks.

## Dataset

The combined corpus contains three sources:

| Source | Domain | Rows |
| --- | --- | ---: |
| `judicial` | Legal | 1,261,948 |
| `pan_Guru` | General | 85,907 |
| `trainclean` | General | 255,705 |
| **Total** |  | **1,603,560** |

The judicial data comes from the [Anuvaad Parallel Corpus](https://github.com/project-anuvaad/anuvaad-parallel-corpus). The preparation script records source, domain, language tags, text lengths, and a stable pair hash for every aligned sentence pair.

### Combined Corpus Summary

- Exact duplicate pairs before cleaning: 2,513
- Average English length: 24.26 words
- Average Punjabi length: 25.97 words
- General-domain rows: 341,612
- Legal-domain rows: 1,261,948

### Cleaning

The three datasets are loaded into a common English-to-Punjabi format and then processed with the following pipeline:

- Normalize text to Unicode NFC and collapse repeated whitespace.
- Remove exact duplicate English-Punjabi pairs using the pair hash.
- Reject punctuation-only and URL-only records.
- Filter extreme sentence lengths and strongly mismatched source/target length ratios.
- Flag very short records and rows containing URLs for review.
- Remove remaining URL and common web-page noise from the training split.
- Retain source and domain labels so performance can be analysed by corpus and domain.

Corpus construction is implemented in [`prepare_parallel_corpus.py`](prepare_parallel_corpus.py), while the cleaning audit and visual analysis are in [`EDA.py`](EDA.py).

## Tokenization

[`tokenization.py`](tokenization.py) trains a shared bilingual byte-pair encoding tokenizer with:

- A 24,000-token vocabulary
- Unicode NFC normalization
- Metaspace whitespace handling
- Translation direction and control tokens
- One vocabulary for English and Gurmukhi text

Sharing the tokenizer allows both translation directions to use the same model vocabulary and makes later student-to-student distillation experiments possible.

## Models

### Sequence-to-Sequence Transformer

[`Gur_slm_seq2seq.py`](Gur_slm_seq2seq.py) implements an encoder-decoder Transformer baseline. It provides the conventional machine-translation setup: the encoder reads the source sentence and the autoregressive decoder generates the target sentence.

### Decoder-Only Transformer

[`gur_slm_decoder.py`](gur_slm_decoder.py) contains the main decoder-only training notebook. Translation direction is represented in the prompt, allowing one causal model to perform both English-to-Punjabi and Punjabi-to-English translation.

The base decoder uses RMSNorm, rotary position embeddings, SwiGLU feed-forward layers, tied token embeddings, mixed-precision training, gradient clipping, and checkpoint resume support. The current base checkpoint has approximately **58.1 million parameters** and is published as [Ajaple/gur-slm-decoder-base](https://huggingface.co/Ajaple/gur-slm-decoder-base).

## Teacher Distillation

[`distillation.py`](distillation.py) implements the teacher-refinement stage. [Sarvam-Translate](https://huggingface.co/sarvamai/sarvam-translate) is used as the English-to-Punjabi teacher because it supports Punjabi and produced stronger translations than the initial student during teacher qualification.

The teacher and student use different tokenizers, so the main training path uses **quality-gated sequence-level knowledge distillation** rather than exact token-level KL divergence:

1. Generate Punjabi translations with the teacher.
2. Reject outputs with script errors, English leakage, implausible length ratios, or other quality failures.
3. Mix accepted teacher targets with gold parallel-corpus targets.
4. Fine-tune the decoder-only student with cross-entropy.
5. Compare the original student, teacher, and distilled checkpoint.

The notebook also includes a MiniLLM-inspired on-policy reverse-KL experiment. Because the vocabularies differ, this is treated as a sequence-level diagnostic and not claimed as exact token-level reverse KL. See [MiniLLM: Knowledge Distillation of Large Language Models](https://arxiv.org/abs/2306.08543).

## Evaluation

Evaluation combines corpus metrics with targeted failure checks:

- BLEU and chrF through SacreBLEU
- Gurmukhi script error rate
- English leakage rate
- Instruction leakage rate
- Side-by-side translation inspection
- A manually reviewed challenge set
- [FLORES+](https://huggingface.co/datasets/openlanguagedata/flores_plus) English (`eng_Latn`) to Punjabi (`pan_Guru`) evaluation

FLORES+ is reserved for evaluation rather than training. Larger `dev` and `devtest` runs are still required before making broad generalization claims from the current small-cache distillation experiment.

## Notebooks and Scripts

| File | Purpose |
| --- | --- |
| [`EDA.py`](EDA.py) | Corpus audit, cleaning, and visualizations |
| [`tokenization.py`](tokenization.py) | Shared bilingual BPE tokenizer training |
| [`Gur_slm_seq2seq.py`](Gur_slm_seq2seq.py) | Sequence-to-sequence Transformer training |
| [`gur_slm_decoder.py`](gur_slm_decoder.py) | Decoder-only Transformer training and analysis |
| [`distillation.py`](distillation.py) | Teacher qualification, sequence KD, reverse-KL diagnostic, and evaluation |
| [`prepare_parallel_corpus.py`](prepare_parallel_corpus.py) | Reproducible three-corpus assembly |

The original online data-analysis notebook is available on [marimo molab](https://molab.marimo.io/notebooks/nb_7UA5TaVaoCqvAZ16d93KKL).

## Future Work

- Run full FLORES+ `dev` and `devtest` evaluations for the base and distilled models.
- Quantize the refined model to 8-bit and 4-bit and measure quality, memory use, and latency.
- Investigate quantization-aware 1.58-bit and 1-bit model variants rather than treating them as simple post-training conversions.
- Export the best practical checkpoint to a mobile-compatible runtime.
- Test translation quality, peak memory, startup time, and tokens per second on real mobile hardware.

## Data License

[![CC BY 4.0][cc-by-shield]][cc-by]

The corpus work is licensed under a [Creative Commons Attribution 4.0 International License][cc-by]. Individual source datasets and teacher checkpoints retain their own licenses and terms; review them before redistributing derived artifacts.

[cc-by]: https://creativecommons.org/licenses/by/4.0/
[cc-by-shield]: https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg
