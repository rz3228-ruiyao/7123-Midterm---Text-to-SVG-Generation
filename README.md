# Text-to-SVG Generation — NYU DL Spring 2026 Midterm

Fine-tuning Qwen2.5-Coder-3B-Instruct with LoRA to generate valid SVG code from natural language prompts.

## Repository Structure

```
├── train_clean.ipynb          # Main training pipeline
├── eval_clean.ipynb           # Inference, post-processing & submission
├── train_rag_clean.ipynb      # RAG-augmented training (experiment)
├── eval_rag_clean.ipynb       # RAG-augmented inference (experiment)
├── requirements.txt           # Python dependencies
└── README.md
```

## Setup

```bash
pip install -r requirements.txt
```

## Training

1. Place `train.csv` (Kaggle competition dataset) in the working directory
2. Open `train_clean.ipynb` and run all cells:
   - Cell 2: Preprocesses SVGs
   - Cells 4–9: Configures LoRA, loads model, trains for 3 epochs

**Training time:** ~7.5 hours on RTX 4080 Super (16GB VRAM)

## Inference

1. Place `test.csv` in the working directory
2. Set `ADAPTER_DIR` in `eval_clean.ipynb` to your trained checkpoint path
3. Run all cells to generate `submission.csv`

**Inference time:** ~2 hours (greedy), ~5 hours (best-of-3)

## Model Weights

Saved in model folder

## Approach

**Base model:** Qwen2.5-Coder-3B-Instruct — a code-specialized LLM within the 4B parameter competition limit.

**Fine-tuning:** LoRA (r=128, alpha=256) applied to all attention and MLP projections. Effective learning rate = 2e-4 (lr=1e-4 × alpha/r=2.0). Cosine scheduler with 100-step warmup.

**Data preprocessing:**
- Scale all SVGs to 256×256 canvas
- Filter by allowed SVG tags (Kaggle whitelist)
- Drop SVGs with transforms or `currentColor`
- Character length filter (≤2500)

**Post-processing pipeline:**
- Fuzzy stutter guillotine (detects and truncates repetitive path commands)
- Smart tag healer (closes truncated SVGs from token limit cutoffs)
- Exact element deduplication (removes duplicate paths)
- CairoSVG rendering fixes (arc flags, empty fills)

**Inference strategies tested:**
- Greedy decoding with two-pass retry (baseline: 14.0 on Kaggle)
- Best-of-3 sampling with pixel variance selection


## Reproducibility

- **Random seed:** 42 (set for Python, NumPy, PyTorch)
- **Hardware:** NVIDIA RTX 4080 Super, 16GB VRAM
- **CUDA:** 12.x
- **Python:** 3.10+
- **Key versions:** See `requirements.txt`

## AI Tooling Disclosure

Claude (Anthropic) was used for coding assistance and debugging.