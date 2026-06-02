# Claude Prompt Evals

This project evaluates how well a Claude model can follow short AWS-focused coding prompts and return syntactically valid outputs in three formats:
- `regex`
- `json`
- `python`

The evaluation pipeline lives in `001_prompt_evals.ipynb`, and the task set is stored in `dataset.json`.

## What the evaluation is doing

For each test case in `dataset.json`, the notebook runs this loop:

1. **Prompt the model to solve the task**
   - `run_prompt(test_case)` sends the task text and asks for only code/regex/JSON.
   - The notebook pre-seeds the assistant with a fenced-code opener (for example ```` ```code ````) and stops when the closing fence appears.

2. **Grade the answer in two ways**
   - **Syntax score** (`grade_syntax`) checks whether the raw output parses/compiles:
     - JSON -> `json.loads(...)`
     - Python -> `ast.parse(...)`
     - Regex -> `re.compile(...)`
   - Syntax score is binary: `10` if valid, `0` if invalid.

   - **Model review score** (`grade_by_model`) asks Claude to act as an AWS reviewer and return JSON with:
     - `strengths`
     - `weaknesses`
     - `reasoning`
     - `score` (1-10)

3. **Compute final score per task**
   - `run_test_case` averages both parts:
   - `final_score = (syntax_score + model_score) / 2`

4. **Aggregate results**
   - `run_eval(dataset)` executes all tasks and prints:
   - average score across test cases
   - Then results are printed as pretty JSON.

## Why this approach is useful

- It measures both **format correctness** (strict syntax validity) and **task quality** (review score).
- It is lightweight and easy to extend with more test cases.
- It works well for prompt iteration: tweak prompt text, rerun, compare scores.

## Eval dataset

Each task includes:
- `task`: natural-language instruction
- `format`: expected output type
- `solution_criteria`: grading guidance used by the reviewer model

## Setup

Create a Python environment and install the small dependency set used in the notebook.

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install anthropic python-dotenv jupyter
```

Create a `.env` file in the project root with your Anthropic key:

```bash
ANTHROPIC_API_KEY=your_api_key_here
```

## Run the evaluation

Open the notebook and run cells top-to-bottom.

```bash
jupyter notebook 001_prompt_evals.ipynb
```

The notebook will:
- generate/write `dataset.json` (or you can keep/edit the existing one)
- run each prompt evaluation
- print per-test outputs and overall average score

