
# Work Field Correlation Matrix

## Background & Objective
danube.ai maintains a curated catalogue of 180 occupational work fields used across its matching and recommendation platform. Each field carries a short code and both German and English labels. This repository generates a similarity-based correlation matrix over these fields and exports it as a JSON file for downstream consumption.

---

## Repository Contents
- `work_fields.json` — input data (original, unmodified)
- `generate_matrix.ipynb` — notebook used to compute and export the matrix
- `correlation_matrix.json` — generated output
- `requirements.txt` — dependencies
- `Explanation for adding duplicates.pdf` — explanation why I added duplicates in the output
- `README.md` — description

---

## Methodology (Implemented)
This solution computes text embeddings for each work field label (German + English), calculates cosine similarities, and exports the **top-10 most similar** fields per work field.

### 1) Text representation
For each entry, a combined text is created:

````
"{nameDe} (DE), {nameEn} (EN)"
````

This keeps both languages and explicitly tags them.

### 2) Embeddings model
Embeddings are produced using Sentence Transformers:

- Model: `sentence-transformers/paraphrase-multilingual-mpnet-base-v2`
- Encoding: `normalize_embeddings=True` (so cosine similarity behaves consistently)

### 3) Similarity computation
A full pairwise cosine similarity matrix is computed:

````
sim_matrix = cosine_similarity(embeddings).astype(np.float64)
````


### 4) Top-K selection and ranking
For each work field `code1`:

- take `top_k = 10` indices by descending similarity (`np.argsort(sims)[::-1][:top_k]`)
- self-similarity is included (the field typically appears with score ≈ 1.0)
- assign integer rank values from **10 down to 1**:
  - `value = 10` for the most similar
  - `value = 1` for the 10th most similar

### 5) Output rows (duplicates by design)
The output stores one top-10 list per work field, meaning the JSON contains:

- exactly `n * 10` objects (for 180 fields → 1800 objects)
- entries can appear in both directions, e.g.:
  - (A → B) and (B → A)

This design makes it easy to retrieve the **top related work fields for any given field directly**.

---

## How to Run (Conda)

### 1) Clone the repository

```bash
git clone https://github.com/ekaanshkhosla/Work-Field-Correlation-Matrix.git
cd Work-Field-Correlation-Matrix
```

### 2) Create and activate an environment

```bash
conda create -n danube-corr python=3.11 -y
conda activate danube-corr
```

### 3) Install dependencies

```bash
pip install -r requirements.txt
```

### 4) Run the generator

Open the notebook and run all cells:

```
generate_matrix.ipynb
```

or start Jupyter with:

```bash
jupyter notebook
```

### 5) Output

After running the notebook, the following file will be generated:

```
correlation_matrix.json
```
