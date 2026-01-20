# CLAUDE.md

Project-specific instructions for Claude Code.

## Project Overview

GitHub Issue Clustering - A Jupyter notebook for semantic clustering of GitHub issues using Gemini embeddings and hierarchical agglomerative clustering. Supports multi-level clustering with interactive visualizations.

## Key Files

- `issue_clustering.ipynb` - Main notebook with all clustering, visualization, and comparison logic
- `requirements.txt` - Python dependencies
- `.env` - API keys (GOOGLE_API_KEY, GITHUB_TOKEN)

## Directory Structure

- `data/` - Cached issue CSVs (gitignored)
- `embeddings/` - Cached numpy embedding files (gitignored)
- `results/` - Clustering output CSVs and hierarchy markdown files (gitignored)

## Common Tasks

### Adding a new repo to analyze
1. Change `REPO_NAME` in the config cell (e.g., `"facebook/react"`)
2. Optionally set `ISSUE_STATE` ("open", "closed", "all") and `MAX_ISSUES`
3. Run all cells - will auto-fetch if CSV doesn't exist

### Force refresh embeddings
Set `FORCE_RECOMPUTE = True` in config cell

### Run hierarchical clustering
Run the "Hierarchical Clustering" section cells - outputs:
- Interactive sunburst/treemap visualizations
- Dendrogram with cut lines at each level
- Markdown file at `results/{repo}_hierarchy.md`

### Compare multiple repos
Run notebook for each repo, then run the comparison cell at the bottom

## Key Functions

- `hierarchical_cluster_issues()` - Main function for multi-level clustering
- `save_hierarchy_markdown()` - Export tree to markdown with issue numbers
- `cluster_issues()` - Single-threshold clustering (original)
- `embed_texts()` - Get embeddings with caching
- `load_or_fetch_issues()` - Load from cache or fetch from GitHub API

## Technical Notes

- Gemini API has 100-item batch limit - both `cluster_issues()` and `embed_texts()` handle this
- Embedding cache uses naming: `{repo_name}__{model_name}.npy`
- CSV cache uses naming: `{repo_name}.csv` (slashes replaced with `__`)
- Hierarchical clustering uses scipy's `linkage` and `fcluster` with average linkage
- Default similarity levels: 50% (loose), 65% (medium), 80% (tight)

## Dependencies

Key packages: sentence-transformers, scikit-learn, scipy, google-genai, umap-learn, plotly, matplotlib, pandas, numpy
