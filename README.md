# Work Field Correlation Matrix (danube.ai TASK-02/2026)

## Background & Objective
danube.ai maintains a curated catalogue of 180 occupational work fields used across its matching and recommendation
platform. Each field carries a short code and both German and English labels. Your task is to derive a meaningful
correlation matrix over these fields, quantifying how similar any two fields are, and export it as a structured JSON file ready
for downstream consumption.


---

## Repository Contents
- `work_fields.json` — input data (original, unmodified)
- `generate_matrix.py` (or `generate_matrix.ipynb`) — script/notebook to compute and export the matrix
- `correlation_matrix.json` — generated output (upper triangle + diagonal)
- `README.md` — this document

---

## Methodology (high level)
This repository computes a similarity score between work fields based on their labels, then converts similarities into ranks.

A typical approach is:
1. Build a text representation for each work field, e.g.  
   `"{nameDe} (DE), {nameEn} (EN)"`
2. Embed texts into vector representations (e.g., Sentence Transformers).
3. Compute pairwise cosine similarity.
4. For each work field:
   - keep the **top-10** most similar (excluding itself for ranking; self is always added back with rank 10)
   - map the ordered list into integer ranks **10..1**
5. Enforce symmetry by **storing only one direction** (upper triangle) in the output JSON.

The detailed choices (model, threshold, text preprocessing, and ranking rules) should be documented in the comments of `generate_matrix.py` and/or in an additional “Methodology” section if you expand this README.

---

## How to Run (Conda)

### 1) Create and activate an environment
```bash
conda create -n danube-corr python=3.11 -y
conda activate danube-corr
```

### 2) Install dependencies
If you have a `requirements.txt`:
```bash
pip install -r requirements.txt
```

If you don’t, a common minimal setup for an embeddings-based solution is:
```bash
pip install numpy scikit-learn sentence-transformers
```

(Optional, only if you run a notebook)
```bash
pip install notebook ipykernel
python -m ipykernel install --user --name danube-corr --display-name "Python (danube-corr)"
```

### 3) Run the generator
```bash
python generate_matrix.py
```

### 4) Output
After running, you should see:
- `correlation_matrix.json` created/updated in the project root.

---

## Output Format
`correlation_matrix.json` is a JSON array where each entry looks like:

```json
{
  "code1": "w_tele",
  "code2": "w_sale",
  "value": 9,
  "score": 0.8123
}
```

Notes:
- Only **upper triangle + diagonal** should be stored (no duplicates).
- `value` is the **rank** (10..1) within the top-10 neighborhood of `code1`.
- `score` is optional but recommended for debugging and transparency.

---

## Reproducibility Notes
- If you use a model with stochastic behavior, set a random seed where applicable.
- Ensure the output is deterministic given the same input and environment.
- Keep `work_fields.json` unchanged.

---

## Troubleshooting
- If you get slow runtime, ensure embeddings are computed once and similarity uses efficient matrix operations.
- If you see non-serializable values in JSON, cast NumPy types to Python types (e.g., `float(score)` and `int(value)`).

---

## License / Use
This repository is intended for candidate assessment submission and local evaluation.
