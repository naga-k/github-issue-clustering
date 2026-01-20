# GitHub Issue Clustering

Analyze and cluster GitHub issues using semantic embeddings. Automatically fetches issues from any GitHub repo, clusters them by similarity, and provides interactive visualizations.

## Features

- **Auto-fetch from GitHub**: Just set `REPO_NAME` and issues are fetched automatically
- **Embedding caching**: Embeddings saved locally to avoid recomputation
- **Multiple clustering strategies**: Agglomerative, vector-style neighbor join, centroid-style
- **Interactive visualizations**: 2D/3D UMAP plots with Plotly
- **Cross-repo comparison**: Analyze multiple repos and compare clustering results
- **Evaluation metrics**: ARI, AMI, Homogeneity, Completeness, V-measure (when labels available)

## Quick Start

1. Create virtual environment:
   ```bash
   python -m venv venv && source venv/bin/activate
   pip install -r requirements.txt
   ```

2. Set up `.env` file:
   ```
   GOOGLE_API_KEY=your_gemini_api_key
   GITHUB_TOKEN=your_github_personal_access_token
   ```

3. Open `issue_clustering.ipynb` and set `REPO_NAME`:
   ```python
   REPO_NAME = "owner/repo"  # e.g., "facebook/react"
   ```

4. Run all cells - issues will be fetched, embedded, clustered, and visualized.

## Directory Structure

```
├── data/                    # Cached issue CSVs (auto-created)
├── embeddings/              # Cached embedding numpy files (auto-created)
├── results/                 # Clustering results for comparison (auto-created)
├── issue_clustering.ipynb   # Main notebook
├── requirements.txt
└── .env                     # API keys (not committed)
```

## Configuration

In the notebook's config cell:

| Variable | Description |
|----------|-------------|
| `REPO_NAME` | GitHub repo to analyze (e.g., `"plotly/plotly.js"`) |
| `FORCE_RECOMPUTE` | Set `True` to recompute embeddings even if cached |
| `SIM_THRESHOLD` | Similarity threshold for clustering (default: 0.72) |

## Workflow

1. **Set repo** → `REPO_NAME = "owner/repo"`
2. **Fetch/load issues** → Auto-downloads if not cached in `data/`
3. **Compute embeddings** → Uses Gemini API, caches in `embeddings/`
4. **Cluster** → Agglomerative clustering with cosine similarity
5. **Visualize** → 2D/3D UMAP plots, similarity heatmaps
6. **Save results** → Stored in `results/` for cross-repo comparison

## Embedding Models

- **Default**: Gemini (`gemini-embedding-001`) - requires `GOOGLE_API_KEY`
- **Offline**: Any SentenceTransformer model (e.g., `all-MiniLM-L6-v2`)

## Notes

- Gemini API has a 100-item batch limit - handled automatically
- Issues are cached locally after first fetch
- NaN/empty values handled safely throughout
