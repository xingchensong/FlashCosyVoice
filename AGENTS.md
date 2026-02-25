# AGENTS.md

## Cursor Cloud specific instructions

### Overview

FlashCosyVoice is a lightweight vLLM-like inference engine for CosyVoice2 text-to-speech, written in pure Python with PyTorch. It is a single Python package (not a monorepo).

### Key caveats

- **CUDA/GPU required for inference.** The full CLI (`flashcosyvoice`) and modules depending on `flash_attn` or `torchaudio` CUDA bindings will not load on a CPU-only VM. The Cloud Agent VM has no GPU, so inference cannot be run end-to-end.
- **No test suite exists.** There are no unit/integration tests in this repo. Validation is limited to linting and import checks.
- **`flash-attn` cannot be pip-installed without CUDA toolkit.** On a CPU-only VM, the package is omitted. Modules that import it (`qwen2`, `model_runner`, `llm_engine`) will fail to import.
- The update script installs torch CPU-only (to avoid CUDA mismatch) then the rest of the deps + the package in editable mode. `flash-attn` is excluded.

### Linting

Linting is configured via pre-commit (`.pre-commit-config.yaml`). Run:

```sh
pre-commit run --all-files
```

Individual linters:
- `flake8 --config=.flake8 flashcosyvoice/`
- `isort --check-only flashcosyvoice/`
- `pydoclint flashcosyvoice/`

Note: flake8 reports some pre-existing warnings (E226, E251, B905, F824). These are in the existing codebase.

### Running the application

On a GPU machine, the CLI is invoked via `torchrun` (see `README.md` "Example Usage"). It requires:
1. Downloaded CosyVoice2-0.5B model weights (from ModelScope)
2. Input JSONL data file with prompt WAV audio paths
3. NVIDIA GPU with CUDA

### Package installation (for development)

```sh
pip install -e .
```

On CPU-only VMs, use `--no-deps` and install dependencies manually (excluding `flash-attn`).

### PATH note

Pip-installed scripts land in `~/.local/bin`. Ensure `PATH` includes it:

```sh
export PATH="$HOME/.local/bin:$PATH"
```
