# Comparing Vector and Graph-Augmented RAG for AML Compliance

Companion repository for the manuscript **“Comparing Vector and Graph-Augmented RAG for AML Compliance: A Question-Type-Aware Evaluation”**.

The repository contains the processed corpus, source manifest, evaluation question bank, vector and graph RAG implementations, graph-construction prompt and graph artifact, FAISS index, raw experimental runs, and consolidated evaluation results used in the study.

## Repository structure

```text
.
├── README.md
├── requirements.txt
├── LICENSE
│
├── data/
│   ├── corpus_sources.csv
│   ├── corpus_chunks.json
│   ├── index/
│   │   └── faiss_index.index
│   └── graph/
│       └── aml_graph.json
│
├── evaluation/
│   └── question_bank.json
│
├── prompts/
│   └── graph_construction_prompt.txt
│
├── notebooks/
│   ├── corpus_preparation_and_chunking.ipynb
│   ├── build_faiss_index.ipynb
│   ├── rag_vectorial.ipynb
│   └── rag_graph.ipynb
│
├── runs/
│   ├── vector_rag_run.json
│   └── graph_rag_run.json
│
└── results/
    ├── per_question_metrics.xlsx
    └── effect_sizes.xlsx
```

## Corpus

The experimental corpus contains **106 source documents** and **1,529 text chunks** related to anti-money laundering and countering the financing of terrorism (AML/CFT).

The source collection combines:

- 31 Spanish Wikipedia entries covering core AML/CFT concepts;
- 75 applied-domain documents from public institutional and regulatory sources.

`data/corpus_sources.csv` is the source manifest. It records the document identifier, title, category, document type, original format, language, retrieval source, URL, and retrieval date.

`data/corpus_chunks.json` contains the processed corpus used by the retrieval systems. Each chunk retains its source metadata and is linked to the corresponding document through `doc_id`.

Documents were segmented into chunks of **up to 800 whitespace-delimited words**, with an overlap of **120 words**.

Original third-party source files are not redistributed in this repository. Source locations are documented in `data/corpus_sources.csv`.

## Vector index

The vector retrieval system uses:

- embedding model: `intfloat/multilingual-e5-base`;
- normalized embeddings;
- `passage:` prefix for corpus chunks;
- `query:` prefix for user questions;
- FAISS `IndexFlatIP`;
- top-k retrieval: `k = 5`.

`data/index/faiss_index.index` is the FAISS index used in the experiment.

The index can also be reconstructed from `data/corpus_chunks.json` using `notebooks/build_faiss_index.ipynb`.

The indexing notebook retains the original variable name `chunks_filtrados`; in the final indexing step, however, no chunks were removed and all 1,529 chunks were indexed.

## Graph artifact

`data/graph/aml_graph.json` is the graph used by the graph-augmented RAG system. It was constructed through LLM-assisted extraction with a controlled ontology.

The graph contains:

- 109 nodes;
- 119 edges.

The prompt used to guide graph construction is provided in:

`prompts/graph_construction_prompt.txt`

Providing the resulting graph directly avoids requiring users to regenerate a potentially non-deterministic LLM-extracted graph before reproducing the downstream experiment.

## Evaluation question bank

`evaluation/question_bank.json` contains the **30 questions** used in the experiment, together with their original metadata, including question type, reasoning complexity, evidence distribution, evaluation objective, and related source documents.

For the article-level analysis, questions were grouped into five categories:

| Category | n |
|---|---:|
| Factual / definitional | 6 |
| Comparative | 4 |
| Relational / explanatory | 6 |
| Procedural / synthesis | 4 |
| Applied / analytical | 10 |
| **Total** | **30** |

The JSON preserves the original, more fine-grained question labels used during experiment preparation.

## RAG systems

### Vector RAG

`notebooks/rag_vectorial.ipynb` implements the dense vector baseline.

Main experimental settings:

- embedding model: `intfloat/multilingual-e5-base`;
- FAISS retrieval: top-5 chunks;
- generator: `llama3.2:3b` through Ollama;
- generation temperature: `0.0`;
- generation context window: `8192`.

The generation prompt instructs the model to answer only from the retrieved context and to state explicitly when the available context is insufficient.

### Graph-augmented RAG

`notebooks/rag_graph.ipynb` implements the graph-augmented system.

Main experimental settings:

- vector chunk retrieval: top-5;
- graph-unit retrieval: top-5;
- graph expansion depth: 1;
- maximum graph units used: 5;
- maximum activated nodes: 8;
- maximum neighboring nodes: 6;
- maximum neighboring edges: 10;
- embedding model: `intfloat/multilingual-e5-base`;
- generator: `llama3.2:3b` through Ollama;
- generation temperature: `0.0`;
- generation context window: `8192`;
- maximum generated tokens: `512`.

The graph system combines evidence from vector-retrieved chunks, semantically retrieved graph units, activated entities, and one-hop graph expansion before constructing the final generation context.

## Raw experimental runs

The `runs/` directory contains the unscored execution records produced by the two RAG systems:

- `vector_rag_run.json`
- `graph_rag_run.json`

These files preserve the retrieval and generation traces for each question, including retrieved material, similarity scores, generated answers, latency information, token counts, and experimental parameters.

They are provided separately from the consolidated result tables so that the original system outputs can be inspected without rerunning the models.

## Evaluation

The study combines multiple evaluation perspectives.

### Human evaluation of retrieval

Retrieval quality was manually assessed on a 0–1 scale using the evidence requirements of the question bank and the retrieved context.

The consolidated per-question scores are included in `results/per_question_metrics.xlsx`. Qualitative working comments used during assessment are not redistributed.

### Expert evaluation of generated answers

Generated answers were evaluated by a domain expert on a 0–1 relevance scale assessing whether each answer was correct, sufficient, and appropriate to the question.

The numerical expert scores used in the analysis are included in `results/per_question_metrics.xlsx`.

Qualitative expert comments are not included in the public repository.

### Automatic LLM-as-judge evaluation

Automatic evaluation is implemented directly in both RAG notebooks.

The evaluator uses `llama3.2:3b` with temperature `0.0` and scores three dimensions from 0.0 to 1.0:

1. context relevance;
2. answer relevance;
3. faithfulness of the answer to the retrieved context.

The procedure is inspired by RAGAS-style dimensions but is implemented as a single custom LLM-as-judge prompt rather than through the RAGAS software package.

For the graph system, the judge evaluates the final context assembled after graph augmentation.

### Efficiency

Both RAG notebooks also record efficiency-related measurements, including:

- total RAG latency;
- retrieval latency;
- generation latency;
- input and output token counts;
- prompt length;
- Ollama timing information.

## Results

`results/per_question_metrics.xlsx` contains the consolidated per-question experimental metrics for both architectures.

`results/effect_sizes.xlsx` contains the effect-size analysis by question category and evaluation metric, including descriptive statistics, mean differences, and Cohen's dz.

The spreadsheets are retained in `.xlsx` format because they are the consolidated analysis files used to prepare the article results.

## Running the notebooks

The notebooks were originally developed and executed in **Google Colab**.

Install the Python dependencies with:

```bash
pip install -r requirements.txt
```

The RAG notebooks additionally require **Ollama** and the local model:

```bash
ollama pull llama3.2:3b
```

The original Colab notebooks install and run Ollama inside the notebook environment.

### Recommended entry points

If the objective is to inspect or reproduce the RAG experiment from the prepared artifacts, the most relevant notebooks are:

1. `notebooks/build_faiss_index.ipynb`
2. `notebooks/rag_vectorial.ipynb`
3. `notebooks/rag_graph.ipynb`

`notebooks/corpus_preparation_and_chunking.ipynb` documents the earlier corpus-construction and chunking workflow. Original downloaded third-party documents are not bundled in the repository.

## Note on original notebook paths and filenames

The notebooks preserve the filenames and `/content` paths used in the original Colab experiment. The public repository uses clearer filenames and folders.

The main correspondence is:

| Original notebook filename | Repository artifact |
|---|---|
| `corpus_aml_chunks.json` / `chunks_filtrados.json` | `data/corpus_chunks.json` |
| `indice_faiss_aml.index` | `data/index/faiss_index.index` |
| `banco_evaluacion_con_docs_rel.json` | `evaluation/question_bank.json` |
| `grafo_aml_llm.json` | `data/graph/aml_graph.json` |

Accordingly, users should either place the repository artifacts under the original filenames expected by the notebooks or update the path variables at the beginning of each notebook.

This section can be removed if the notebooks are later adapted to the final repository structure.

## Dependency versions

The original notebooks installed their Python dependencies without pinning package versions. For that reason, `requirements.txt` lists the direct dependencies but does not claim exact historical package versions that were not recorded.

## Reproducibility note

The repository preserves the concrete FAISS index, graph artifact, question bank, raw RAG runs, and consolidated results used in the experiment.

Regenerating LLM outputs or rebuilding the graph may not produce byte-identical results because model execution and LLM-assisted extraction can vary across environments and software/model versions. The preserved artifacts are therefore the reference outputs for the reported experiment.

## Data provenance and third-party material

The source manifest identifies the public sources from which the experimental corpus was assembled.

This repository does **not** claim ownership of third-party source texts. Rights and reuse conditions associated with source material remain with their respective rightsholders and source providers.

See `LICENSE` for the current repository licensing notice.

## Citation

Citation information will be added when the associated article has its final bibliographic record and/or DOI.

If you use this repository before that information is available, please cite the associated manuscript and identify the repository version used.
