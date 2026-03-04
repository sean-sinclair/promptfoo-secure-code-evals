# Copilot Instructions

## Project Overview

This project uses [promptfoo](https://www.promptfoo.dev/) to evaluate whether prompt phrasing affects the security quality of LLM-generated code. The core finding: prefixing prompts with `"Using best practices:"` improves security pass rates by ~16.5%.

Two parallel prompt sets are evaluated:
- **Regular set** (`prompts/regular.txt`): 200 plain coding prompts
- **Secure set** (`prompts/secure.txt`): Same 200 prompts, each prefixed with `"Using best practices: "`

## Commands

### Running evaluations

```sh
# Run a full evaluation (--repeat N for multiple runs, -j N for parallelism)
promptfoo eval --repeat 3 -j 6 -c ./configs/conf_std_4omini.yaml

# Run a quick small test
promptfoo eval -c ./configs/conf_small_test.yaml

# View results in the UI
promptfoo view
```

### Python app (uses uv)

```sh
cd app
uv run python main.py
```

## Architecture

```
configs/     # promptfoo YAML configs — each references a prompt file and a model
prompts/     # Prompt sets as .txt files — entries separated by ---
.app/         # Python utility (uses uv + pyproject.toml)
```

**Config naming convention**: `conf_{std|sec|demo}_{model}.yaml`
- `std` = standard/regular prompts, `sec` = "using best practices" prompts
- Model suffix: `4omini`, `51`, `5mini`

**All configs share the same `llm-rubric` assertion** to evaluate generated code for security issues (no hardcoded secrets, parameterized queries, secure API connections, least privilege).

## Key Conventions

- **Prompt files** use `---` as the delimiter between individual test prompts.
- **Config YAML files** reference prompt files with relative paths: `file://../prompts/filename.txt`
- **Secure prompt variants** are created by prepending `"Using best practices: "` to each prompt — do not alter the core prompt text otherwise.
- **API keys** go in `.env` at the repo root; `OPENROUTER_API_KEY` is required. The file is gitignored — never commit secrets.
- When adding a new model to test, create two config files: one for `regular.txt` and one for `secure.txt`, following the existing naming pattern.
- The `conf_demo*.yaml` configs use a smaller 10-prompt set (`demo.txt` / `demo_secure.txt`) for quick demonstrations.
