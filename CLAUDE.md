# CLAUDE.md

Project-specific instructions for Claude Code.

## Project Overview

GitHub Issue Clustering - A Jupyter notebook for semantic clustering of GitHub issues using embeddings and agglomerative clustering.

## Key Files

- `issue_clustering.ipynb` - Main notebook with all clustering, visualization, and comparison logic
- `requirements.txt` - Python dependencies
- `.env` - API keys (GOOGLE_API_KEY, GITHUB_TOKEN)

## Directory Structure

- `data/` - Cached issue CSVs (gitignored)
- `embeddings/` - Cached numpy embedding files (gitignored)
- `results/` - Clustering output CSVs (gitignored)

## Common Tasks

### Adding a new repo to analyze
1. Change `REPO_NAME` in the config cell
2. Run all cells - will auto-fetch if CSV doesn't exist

### Force refresh embeddings
Set `FORCE_RECOMPUTE = True` in config cell

### Compare multiple repos
Run notebook for each repo, then run the comparison cell at the bottom

## Technical Notes

- Gemini API has 100-item batch limit - both `cluster_issues()` and `embed_texts()` handle this with batching
- Embedding cache uses naming: `{repo_name}__{model_name}.npy`
- CSV cache uses naming: `{repo_name}.csv` (slashes replaced with `__`)

## Dependencies

Key packages: sentence-transformers, scikit-learn, google-genai, umap-learn, plotly, pandas, numpy
