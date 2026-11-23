# Synapt on GraphRAG-Bench (Medical and Novel)

**Link:** https://www.synapt.ai/

Synapt is Prodapt's enterprise AI suite, built to be grounded, governed and auditable. Its context layer answers
questions over a knowledge graph built from an organization's documents and data, and that is what this benchmark
measures.

This repository holds the results of Synapt on [GraphRAG-Bench](https://graphrag-bench.github.io/) for both subsets
(Medical and Novel), across all four question types. For every question it includes the generated answer, the
retrieved context the answer came from, and the per-question scores from the official evaluator. The answers can be
re-scored with the official, unmodified evaluation code (see [section 5](#5-re-scoring-our-answers)).

## 1. Results

Generation `gpt-4o-mini` at temperature 0.7; judge `gpt-4o-mini`; official evaluation code (commit `fdbab59`),
unmodified; all questions of the official question files. Scores are percentages. **Average** is the unweighted mean
of the four ACC (`answer_correctness`) values, as on the GraphRAG-Bench leaderboard.

### Medical: Average 76.61

| Question type        |       ACC | ROUGE-L | Coverage | Faithfulness | Scored / answered |
| -------------------- | --------: | ------: | -------: | -----------: | ----------------: |
| Fact Retrieval       |     75.87 |   46.51 |        – |            – |     1,096 / 1,098 |
| Complex Reasoning    |     76.96 |   33.71 |        – |            – |         509 / 509 |
| Contextual Summarize |     79.89 |       – |    62.26 |            – |         289 / 289 |
| Creative Generation  |     73.72 |       – |    46.53 |        53.69 |         166 / 166 |
| **Average**          | **76.61** |         |          |              | **2,060 / 2,062** |

### Novel: Average 56.34

| Question type        |       ACC | ROUGE-L | Coverage | Faithfulness | Scored / answered |
| -------------------- | --------: | ------: | -------: | -----------: | ----------------: |
| Fact Retrieval       |     52.13 |   37.67 |        – |            – |         955 / 971 |
| Complex Reasoning    |     52.85 |   26.50 |        – |            – |         606 / 610 |
| Contextual Summarize |     64.85 |       – |    40.02 |            – |         360 / 362 |
| Creative Generation  |     55.54 |       – |    35.64 |        34.12 |           67 / 67 |
| **Average**          | **56.34** |         |          |              | **1,988 / 2,010** |

Every question was answered. "Scored" counts the questions the official evaluator produced a score for; the rest
were dropped by the evaluator itself (see [section 6](#6-counts-and-caveats)).

### Against the published leaderboard

Other systems' figures are taken from the [GraphRAG-Bench leaderboard](https://graphrag-bench.github.io/) as
published; we did not re-run them. All use `gpt-4o-mini` for generation. Sorted by Medical Average.

| Method                       |   Medical |     Novel |
| ---------------------------- | --------: | --------: |
| KnowCosmos                   |     80.15 |     70.34 |
| GraphRAG-SDK                 |     76.87 |     66.09 |
| KT-MOSAIC                    |     76.80 |     64.28 |
| **Synapt (this repository)** | **76.61** | **56.34** |
| GLM-RAG                      |     76.01 |     60.59 |
| Sapient-Bodhi-GraphRAG       |     75.75 |     66.30 |
| AtlasGraphRAG                |     75.62 |     66.18 |
| G-reasoner                   |     73.30 |     58.94 |
| AutoPrunedRetriever-llm      |     67.00 |     63.72 |
| HippoRAG2                    |     64.85 |     56.48 |
| Fast-GraphRAG                |     64.12 |     52.02 |
| LightRAG                     |     62.59 |     45.09 |
| RAG (w/ rerank)              |     62.43 |     48.35 |
| RAG (w/o rerank)             |     61.00 |     47.93 |
| HippoRAG                     |     59.08 |     44.75 |
| StructRAG                    |     58.56 |     49.13 |
| RAPTOR                       |     57.10 |     43.24 |
| Lazy-GraphRAG                |     56.89 |     50.59 |
| KGP                          |     56.33 |     48.01 |
| KET-RAG                      |     47.05 |     47.62 |
| MS-GraphRAG (local)          |     45.16 |     50.93 |
| MS-GraphRAG (global)         |     28.56 |     44.52 |

## 2. System description

Synapt stores each corpus as a knowledge graph (entities, relationships and the text chunks they come from) alongside
a chunk store with vector and full-text indexes. For each question, dense vector search and full-text search find
anchor nodes in the graph, fused with Reciprocal Rank Fusion. The graph is expanded from these anchors, to a depth
chosen by the question's intent. The most relevant text chunks, from the graph and the chunk store, are passed with the
graph context to `gpt-4o-mini`, which writes the answer in a single call. Each question is answered independently,
with no conversation history.

## 3. Configuration of the reported runs

| Item                   | Value                                                                                                                                                                             |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Answer generation      | `gpt-4o-mini` (OpenAI API), temperature 0.7, max_tokens 2048, seed 0, response caching off                                                                                        |
| Context per question   | 10 text chunks, plus graph context                                                                                                                                                |
| Graph expansion        | Depth chosen per question by intent                                                                                                                                               |
| Stored chunk size      | Medical: about 2,700 characters on average; Novel: about 15,600 characters on average (maximum 16,000)                                                                            |
| Embeddings (retrieval) | `text-embedding-ada-002` (1,536 dimensions)                                                                                                                                       |
| Load                   | 10 questions in parallel; 120 s limit per question; questions that errored were re-run until all were answered                                                                    |
| Evaluation             | `Evaluation/generation_eval.py` at commit `fdbab59`, unmodified; `--mode API --model gpt-4o-mini --embedding_model BAAI/bge-large-en-v1.5 --detailed_output`; judge temperature 0 |
| Runs                   | One full run per subset                                                                                                                                                           |

## 4. Files

All result files are in the `graphrag_bench_submission/` folder:

| Path                                                            | Contents                                                                            |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `graphrag_bench_submission/medical/generated_answers.json.xz`   | All 2,062 Medical answers, in the official evaluator's input format (xz-compressed) |
| `graphrag_bench_submission/medical/per_question_scores.json.xz` | The official evaluator's score for each Medical question (xz-compressed)            |
| `graphrag_bench_submission/medical/summary.json`                | Medical per-type scores, other metrics and scored counts                            |
| `graphrag_bench_submission/novel/generated_answers.json.xz`     | All 2,010 Novel answers, in the official evaluator's input format (xz-compressed)   |
| `graphrag_bench_submission/novel/per_question_scores.json.xz`   | The official evaluator's score for each Novel question (xz-compressed)              |
| `graphrag_bench_submission/novel/summary.json`                  | Novel per-type scores, other metrics and scored counts                              |

The answer and score files are compressed with xz to stay within GitHub's file size limit (uncompressed, the Medical
files are about 58 MB each and the Novel files about 310 MB each). The retrieved chunks repeat across questions, so xz
reduces every file to under 4 MB. To decompress (the original `.json` is written next to each file):

```bash
xz -dk graphrag_bench_submission/medical/*.json.xz graphrag_bench_submission/novel/*.json.xz
```

or read them directly in Python with `lzma.open(path, "rt", encoding="utf-8")`.

**`generated_answers.json`**: a JSON list, one object per question, in question-file order.

| Field                                                                   | Meaning                                                 |
| ----------------------------------------------------------------------- | ------------------------------------------------------- |
| `id`, `question`, `source`, `question_type`, `evidence`, `ground_truth` | Copied from the official question file                  |
| `context`                                                               | The retrieved text chunks the answer was generated from |
| `generated_answer`                                                      | Synapt's answer (the text that is scored)               |

**`per_question_scores.json`**: a JSON list, one object per question.

| Field                                                                             | Meaning                                                                                                                                                              |
| --------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `id`, `question_type`, `question`, `ground_truth`, `generated_answer`, `contexts` | The scored sample                                                                                                                                                    |
| `metrics`                                                                         | The evaluator's metrics for the question, as fractions from 0 to 1 (e.g. `answer_correctness`, `rouge_score`, `coverage_score`, `faithfulness`); `null` when dropped |
| `status`                                                                          | `scored`, or `dropped` when the evaluator could not score the question                                                                                               |

**`summary.json`**: per-type `answer_correctness` and the other metrics (as percentages), the `scored` and
`answers` counts, and the `average`.

## 5. Re-scoring our answers

Once decompressed (section 4), the answer files can be passed directly to the official evaluator. In a checkout of
[GraphRAG-Bench/GraphRAG-Benchmark](https://github.com/GraphRAG-Bench/GraphRAG-Benchmark) at commit `fdbab59`:

```bash
export LLM_API_KEY=<your_openai_key>

python -m Evaluation.generation_eval \
  --mode API \
  --model gpt-4o-mini \
  --embedding_model BAAI/bge-large-en-v1.5 \
  --data_file <path_to_this_repo>/graphrag_bench_submission/medical/generated_answers.json \
  --output_file scores_medical.json \
  --detailed_output
```

For the Novel subset, pass `graphrag_bench_submission/novel/generated_answers.json`. The judge's output varies a
little between runs, so re-scored numbers can differ slightly from ours, and a few questions may be dropped in
different places (see below).

## 6. Counts and caveats

|                                                           | Medical | Novel |
| --------------------------------------------------------- | ------: | ----: |
| Questions in the official file                            |   2,062 | 2,010 |
| Answers produced                                          |   2,062 | 2,010 |
| Generation errors or empty answers in the published files |       0 |     0 |
| Questions scored by the evaluator                         |   2,060 | 1,988 |

- **Dropped questions.** For some answers, the judge returns its list of statements as plain strings instead of
  `{statement, reason}` objects. The evaluator's `ClassificationWithReason` validation rejects the reply, the error is
  not caught by the metric's fallback, and the question is left out of the average with no score. Dropped questions
  were re-scored twice with the unmodified evaluator; those still dropped (2 Medical, 22 Novel) are excluded, as the
  evaluator does.
- **Duplicate question id.** The Novel question file contains the id `Novel-55f0c0e2` twice: the same question text
  under two different `source` values, with two different reference answers. Our runner stores answers by id, so both
  rows carry the same answer and both are scored against the same reference. We kept both rows as the file has them.
- **Single run.** Each subset was run once at temperature 0.7, so a repeat run would give a slightly different
  Average.
