# Issue Clustering (CSV workflow)

Cluster and evaluate issue/feedback items stored in `issues_raw.csv`.

## Quick start
1) Create env: `python -m venv .venv && source .venv/bin/activate && python -m pip install --upgrade pip`
2) Install deps: `pip install -r requirements.txt`
3) Put your data in `issues_raw.csv` with columns `title`, `body`, and optional `label`.
4) Set `GOOGLE_API_KEY` in `.env` (or env var) to use Gemini embeddings; otherwise set `model_name` to a SentenceTransformer in the notebook.
5) Open `issue_clustering.ipynb` and run the cells.

## What the notebook does
- Loads issues from CSV.
- Clusters with agglomerative (cosine threshold) and shows a tabular view.
- Evaluates against `label` if present (ARI/AMI/H/C/V).
- Compares two incremental strategies: vector-style neighbor join and centroid-style incremental assignment.
- Includes a threshold sweep to see how hyperparameters affect ARI/AMI.
- **Vector visualization**: Interactive 2D/3D plots using UMAP dimensionality reduction to explore embeddings.
- **Similarity exploration**: Find most similar items and visualize pairwise similarity heatmaps.

## Notes
- Data normalization handles NaN/float inputs safely.
- Gemini is default; to run offline, set `model_name` to a HF model like `sentence-transformers/all-MiniLM-L6-v2`.
- Thresholds: agglomerative uses `sim_threshold`, vector-style uses `threshold`, centroid-style uses its own `threshold`; adjust in the comparison cells.
