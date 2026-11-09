# Explainable Prompt Refinement for Python Code Generation

Research notebooks for XAICoder: A Multi-Agent Framework for XAI-Guided Iterative Prompt Refinement in LLM Code Generation.

The repository includes a combined agentic workflow, separate ablation-study variants, sensitivity experiments, and result-analysis notebooks. It is a notebook-based research implementation, not a packaged library.

## Notebooks

| Notebook | Purpose |
| --- | --- |
| [overal-performance.ipynb] | main workflow of XAICoder. Records performance and execution metadata. |
| [prompt-ablation-only.ipynb] | Prompt-ablation experiment variant for studying the contribution of semantic prompt ablation. |
| [counterfactual-only.ipynb] | Counterfactual-refinement experiment variant for studying the contribution of  Counterfactual-refinement.|
| [sensitivity-analysis.ipynb] | Feature-ablation and perturbation experiments for sensetivity analysis. |
| [evaluation.ipynb] | Analysis of saved experiment outputs, including test success, ablation studies, ambiguity, parameter studies, attribution coherence, and output changes. |



LangGraph coordinates the generation, testing, and explanation/refinement agents. Stopping conditions and iteration limits are defined in each notebook.


## Getting started

### 1. Choose an environment

Google Colab is the most direct starting point: the notebooks use `/content/` paths and Colab-specific download utilities.

Local Jupyter execution requires adapting those paths and replacing or skipping `google.colab` utilities. A GPU may help with local transformer-based components; hosted model inference also requires network access and valid provider credentials.

### 2. Install dependencies

Start with the dependency cells in the selected notebook. 


### 3. Configure credentials and models

Configure only the credentials needed for your selected backend and experiment:

| Variable | Use |
| --- | --- |
| `OPENAI_API_KEY` | OpenAI generation and/or embedding calls where configured. |
| `HF_TOKEN` | Hugging Face inference and any gated resources used by the experiment. |
| `TELEGRAM_BOT_TOKEN` | Optional delivery of result files through Telegram. |
| `TELEGRAM_CHAT_ID` | Destination for optional Telegram delivery. |

The combined notebook also exposes environment-variable settings for other provider credentials; these are not required unless the corresponding integration is used.


In the generation workflows, review these settings:

```python
FLAG = "mbpp"                  # Dataset selection
FLAG_TELE = "False"            # Sending results to Telegram bot
FLAG_Model = "hugging_face"    # API selection
```

The combined notebook uses `"humaneval"` for HumanEval and the existing spelling `"opeiai"` for the OpenAI backend. 

Review `MODEL`, API base URLs, and embedding-model configuration. Model identifiers in the notebooks are experiment settings, not guarantees of current provider availability. 

Telegram delivery is optional. In addition to setting `FLAG_TELE = "False"`, skip or remove any unconditional final `send_dataframe_to_telegram(...)` calls if you do not want uploads.

### 4. Prepare the data

The main generation workflows support:

- **MBPP:** a local JSON Lines file, configured as `/content/MBPP.jsonl`, with task text in `text` and tests in `test_list`.
- **HumanEval:** the Hugging Face `openai/openai_humaneval` test split, loaded through a parquet URL, with task text in `prompt` and tests in `test`.

### Dataset sources

The datasets used in this study were obtained from the following sources:

- **HumanEval:** [openai/openai_humaneval on Hugging Face](https://huggingface.co/datasets/openai/openai_humaneval).
- **MBPP:** [mbpp.jsonl in the Google Research repository](https://github.com/google-research/google-research/blob/master/mbpp/mbpp.jsonl). This study uses the version containing 974 problems. Download this file and configure its local path (the notebooks expect `/content/MBPP.jsonl`).

For reproducibility, record the dataset revision or commit and the download date; these links alone do not pin the exact dataset snapshot used in an experiment.

Retain any additional benchmark fields used by the selected notebook. Review dataset selection, filtering, and slicing cells before running; a configured subset is not necessarily the entire benchmark.

The sensitivity notebook has its own dataset and output configuration. The evaluation notebook consumes experiment CSV files rather than automatically running the generation notebooks.

### 5. Run experiments

1. Open the notebook for the desired experiment.
2. Review authentication, dependencies, dataset loading, and model configuration.
3. Set output paths and iteration parameters.
4. Run a small subset first to validate API access, test execution, and CSV output.
5. Execute the intended experiment cells in order.
6. Save the output files and configuration before starting another run.

The generation notebooks contain example and demonstration cells as well as the experiment workflow. Review these before using **Run all**; some may invoke models or execute code.

Use distinct output filenames for different datasets, models, and experiment variants. Several writers append rows, so rerunning against an existing output file can mix or duplicate results.

## Results and evaluation

Generation workflows write CSV results, commonly to `/content/outputs.csv`, and can record errors separately in `/content/error.csv`. The sensitivity workflow uses its own output filename.

Depending on the notebook, recorded fields include:

- Original and revised prompts, generated code, tests, and test outcomes.
- Refinement cycles and counterfactual outcomes.
- Prompt chunks, ablation results, and attribution-related values.
- Token consumption, API-call counts, runtimes, and provider fingerprints when available.
- Prompt edit distances and embedding-based similarity.
- Code-similarity and sensitivity measurements in the dedicated analysis workflow.

To analyze results, open `evaluation.ipynb`, update its input-file variables, and run the sections relevant to your experiment. Different sections expect different CSV schemas; a single output file is not necessarily suitable for every analysis.

The evaluation notebook includes pass@k-related and round-based summaries. Inspect the implemented calculations and sampling assumptions before reporting them as a particular benchmark estimator. Failed or skipped CSV rows should also be accounted for when interpreting aggregate results.

## Reproducibility

For each run, record:

- Notebook version or Git commit.
- Dataset version, selected rows, and preprocessing.
- Provider, model identifier, generation settings, and embedding model.
- Chunking strategy, perturbation settings, and iteration limits.
- Random seeds where supported.
- Dependency versions and execution environment.
- Output filenames, failed rows, token usage, and API costs where available.

Hosted models, stochastic generation, and provider updates can affect results even when notebook code is unchanged. The cleaned notebooks have undergone static checks, but full end-to-end execution across all providers and datasets has not been verified. No benchmark scores are claimed by this README.

## Security and publication checklist

> **Generated Python code is untrusted.** Test helpers execute generated code. Run experiments in a disposable, isolated environment with resource limits and without sensitive files or unnecessary credentials. These notebooks should not be treated as a secure execution sandbox.



## Repository scope and licensing

This collection is intended for research on XAICoder: A Multi-Agent Framework for XAI-Guided Iterative Prompt Refinement in LLM Code Generation. It does not include a guaranteed reproducible environment, a production deployment interface, or a verified set of benchmark results.

## Counterfactual prompt-generation frameworks

The following counterfactual prompt-generation templates were evaluated in the paper.

### 1. Baseline counterfactual generation

````text
You are an expert prompt engineer specializing in code generation refinement.
## Context
A code generation attempt has FAILED.
## Your Task
Generate EXACTLY 5 counterfactual prompts that minimally edit the original prompt
to generate a better code and fix the failure.
1. **Apply different repair strategies**
2. **Be diverse from each other** (avoid near-duplicates)
##original prompt
{prompt}
## Return exactly 5 prompts using this format:
<new>
Counterfactual 1
Repair Strategy:brief strategy description
Counterfactual Prompt:
```python
the complete counterfactual prompt text
```
</new>
REMEMBER you MUST ENCLOSE each counterfactual prompt within <new> tags.
````

### 2. Quantitative XAI-guided counterfactual generation

````text
You are an expert prompt engineer specializing in code generation refinement.
## Context
A code generation attempt has FAILED. We have performed feature ablation analysis
on the original prompt and identified which semantic chunks are most responsible
for the failure.
## Original Prompt (3 chunks)
Chunk 1: {chunk [0]}
Chunk 2: {chunk [1]}
Chunk 3: {chunk [2]}
## Importance Scores
- similarity; higher = more impact on failure)
- Chunk 1: {ablation_values [0]}
- Chunk 2: {ablation_values [1]}
- Chunk 3: {ablation_values [2]}
## Your Task
Generate EXACTLY 5 counterfactual prompts that minimally edit the original prompt
to generate a better code and fix the failure.Each counterfactual should:
1. **Target high-importance chunks** based on the scores above
2. **Apply different repair strategies**
3. **Be diverse from each other** (avoid near-duplicates)
## Return exactly 5 prompts using this format:
<new>
Counterfactual 1
Target Chunk 1 (Critical)
Repair Strategy:brief strategy description
Counterfactual Prompt:
```python
the complete counterfactual prompt text
```
</new>
REMEMBER you MUST ENCLOSE each counterfactual prompt within <new> tags.
````

### 3. Aggregated XAI-guided counterfactual generation

````text
You are an expert prompt engineer specializing in code generation refinement.
## Context
A code generation attempt has FAILED. split the prompt into 3 chunks. we calculate which chunk is more important in generating the failed code.
## Original Prompt (3 chunks)
Chunk 1: {chunks [0]}
Chunk 2: {chunks [1]}
Chunk 3: {chunks [2]}
## Importance of chunks
- Chunk 1: {ablation_qualities [0]} Impact
- Chunk 2: {ablation_qualities [1]} Impact
- Chunk 3: {ablation_qualities [2]} Impact
## Your Task
Generate EXACTLY 5 counterfactual prompts that edit the original prompt minimal as possible
to generate a better code and fix the failure. Each counterfactual should:
1. **Target high-impact chunks** based on the impacts above
2. **Apply different repair strategies**
3. **Be diverse from each other** (avoid near-duplicates)
## Return exactly 5 prompts using this format:
<new>
Counterfactual 1
Target Chunk 1 (Critical)
Repair Strategy:brief strategy description
Counterfactual Prompt:
```python
the complete counterfactual prompt text
```
</new>
REMEMBER you MUST ENCLOSE each counterfactual prompt within <new> tags.
````

##  Agent prompts and explanation templates

This section presents the supplied agent prompt templates and the explanation templates employed by the proposed framework. 

### 1. Coder agent

**System message**

```text
You are a code generation model tasked with generating code for this instruction.You must provide a complete and correct logic to solve the problem.Do not return a placeholder like `pass` or `#todo`.The implementation must be functional.When the function complete stop .do not generate extra space or newline .generate the full function and then stop.
```

**User message** (rendered template; `{state["prompt"]}` denotes the task prompt)

```text
here is your task: {state["prompt"]}
```

Benchmark tests are retained for evaluation, not included in the code-generation task prompt.

### 2. XAI-Guided Prompt Optimizer Agent

````text
You are an expert prompt engineer specializing in code generation refinement.
## Context
A code generation attempt has FAILED. We have performed feature ablation analysis
on the original prompt and identified which semantic chunks are most responsible
for the failure.
## Original Prompt (3 chunks)
Chunk 1: {chunk [0]}
Chunk 2: {chunk [1]}
Chunk 3: {chunk [2]}
## Importance Scores
- similarity; higher = more impact on failure
- Chunk 1: {ablation_values [0]}
- Chunk 2: {ablation_values [1]}
- Chunk 3: {ablation_values [2]}
## Your Task
Generate EXACTLY 5 counterfactual prompts that minimally edit the original prompt
to generate a better code and fix the failure.Each counterfactual should:
1. **Target high-importance chunks** based on the scores above
2. **Apply different repair strategies**
3. **Be diverse from each other** (avoid near-duplicates)
## Return exactly 5 prompts using this format:
<new>
Counterfactual 1
Target Chunk 1 (Critical)
Repair Strategy: brief strategy description
Counterfactual Prompt:
```python
the complete counterfactual prompt text
```
</new>
REMEMBER you MUST ENCLOSE each counterfactual prompt within <new> tags.
````

### 3. Explanation: failure resolved by removing a harmful chunk

```text
Here's what we learned from analyzing your original prompt:
We split your prompt into 3 meaningful parts and tested what happens when we remove each one:
Chunk 1: {input_examples1 [0]}
Chunk 2: {input_examples1 [1]}
Chunk 3: {input_examples1 [2]}
by removing chunk number {chunk_number} from the original prompt the error would solve.
This means:
this part is a hurmful chunk and it is responsible for the code going wrong.
Removing it caused generating the correct code.
this is the correct prompt:
{ablated_features [chunk_number-1]}
this is the correct code:
{ablated_code}
```

### B.4. Explanation: failure resolved by counterfactual generation

```text
Here's what we learned from analyzing your original prompt:
We split your prompt into 3 meaningful parts and tested what happens when we remove each one:
Chunk 1: {chunk [0]}
Chunk 2: {chunk [1]}
Chunk 3: {chunk [2]}
Importance scores (how much removing that chunk changes the code that gets generated):
Chunk 1: {ablation_values [0]} influential
Chunk 2: {ablation_values [1]} influential
Chunk 3: {ablation_values [2]} influential
This means:
The part with the highest value is the one most responsible for the code going wrong.
Removing it caused the biggest change in behavior (usually for the worse).
Based on this the error message we saw , we created 5 slightly improved version of your prompt.to flip the result and pass the test case.
New (counterfactual) prompts we recommend trying:
{counterfactual_prompt}
the semantic similarity between the original prompt and counterfacual prompt is {sim}
the edit distance between the original prompt and counterfacual prompt is {edit_dis}
The new prompt focuses most fixes on the most important chunk(s) usually just 1 or 2 small changes like clearer wording, added constraints, or an extra example.
the final prompt is :
{prompt}
and this is the final code:
{complete_code}
```


