# Clustering Improvement Options

## Problem Statement
The current clustering uses a fixed similarity threshold (0.72) that doesn't work universally:
- **Plotly (500 issues)**: 0.72 is too tight → 182 clusters, 72 singletons (14% orphaned)
- **Soulcaster (87 issues)**: 0.72 produces 58 clusters → most clusters are tiny (1-2 items)

Each customer/repo has different characteristics. No single threshold works for all.

---

## Options Comparison

| Approach | Complexity | User Effort | Adaptability | Best For |
|----------|------------|-------------|--------------|----------|
| Hierarchical | Medium | Low (explore) | High | Exploration, varied needs |
| Auto-tuning | Medium | None | High | Hands-off users |
| User Slider | Low | Medium | Manual | Power users |
| Adaptive Defaults | Low | None | Medium | Quick improvement |
| Feedback Loop | High | Medium | Very High | Long-term product |

---

## Option 1: Hierarchical Clustering

### How it works
Instead of picking ONE threshold, compute clusters at multiple levels and let users navigate.

```
📁 Visualization Issues (50 issues)        ← Level 1 (loose)
   📁 Hover/Tooltip (20 issues)            ← Level 2 (medium)
      📁 Tooltip positioning (8)           ← Level 3 (tight)
      📁 Tooltip content (7)
      📁 Unified hover (5)
   📁 Legend (15 issues)
   📁 Axis labels (15 issues)
```

### Implementation
- Use `AgglomerativeClustering` with `distance_threshold=None` and `n_clusters=None` to get full dendrogram
- Cut dendrogram at 3-4 levels (e.g., 0.5, 0.65, 0.8 similarity)
- Store parent-child relationships between levels
- UI shows expandable/collapsible tree

### Pros
- Users can zoom in/out to granularity they need
- No "wrong" threshold - all levels available
- Great for exploration and understanding

### Cons
- More complex UI needed
- More data to store/compute
- Users might get overwhelmed by options

---

## Option 2: Auto-tuning (Silhouette Score)

### How it works
Automatically find the "best" threshold for each dataset using cluster quality metrics.

```python
from sklearn.metrics import silhouette_score

def find_optimal_threshold(embeddings):
    best_score = -1
    best_threshold = 0.7

    for threshold in [0.5, 0.55, 0.6, 0.65, 0.7, 0.75, 0.8, 0.85]:
        labels = cluster_at_threshold(embeddings, threshold)

        # Skip if too few clusters or all singletons
        n_clusters = len(set(labels))
        if n_clusters < 2 or n_clusters > len(labels) * 0.8:
            continue

        score = silhouette_score(embeddings, labels, metric='cosine')
        if score > best_score:
            best_score = score
            best_threshold = threshold

    return best_threshold
```

### Implementation
- Before clustering, run threshold sweep (10-15 values)
- Compute silhouette score for each (measures: tight within clusters, separated between)
- Pick threshold with highest score
- Cache result per repo (re-compute on significant data change)

### Pros
- Zero user configuration
- Mathematically grounded
- Adapts to any dataset automatically

### Cons
- Silhouette score isn't perfect - may not match human intuition
- Extra computation time (though can be cached)
- Less user control

---

## Option 3: User-Controlled Slider

### How it works
Simple UI slider that lets users adjust cluster tightness in real-time.

```
Cluster Tightness:
[Fewer, larger clusters] ●━━━━━━━━━━━○ [More, smaller clusters]
                              ↑
                          Current: 0.72
```

### Implementation
- Add threshold parameter to clustering API
- Frontend slider triggers re-clustering on change
- Debounce to avoid excessive API calls
- Show cluster count preview as user drags

### Pros
- Simple to implement
- Users have full control
- Intuitive mental model

### Cons
- Requires user effort/knowledge
- Users may not know what's "good"
- Different per-repo, no memory

---

## Option 4: Adaptive Defaults

### How it works
Rule-based threshold selection based on dataset characteristics.

```python
def get_adaptive_threshold(issues, embeddings):
    n_issues = len(issues)

    # Compute embedding spread (how diverse are the issues?)
    pairwise_sims = cosine_similarity(embeddings)
    avg_similarity = pairwise_sims.mean()

    # Base threshold on dataset size and diversity
    if n_issues < 50:
        base = 0.65  # Small repos: looser
    elif n_issues < 200:
        base = 0.70
    else:
        base = 0.75  # Large repos: tighter

    # Adjust for embedding diversity
    if avg_similarity > 0.6:  # Issues are similar overall
        base -= 0.05  # Be looser
    elif avg_similarity < 0.4:  # Issues are diverse
        base += 0.05  # Be tighter

    return base
```

### Implementation
- Compute dataset stats before clustering
- Apply heuristic rules to select threshold
- Can be tuned based on observed results

### Pros
- No user effort
- Simple to implement
- Easy to understand and debug

### Cons
- Rules are heuristics, not guaranteed optimal
- May need tuning over time
- Less precise than auto-tuning

---

## Option 5: Feedback Loop / User Corrections

### How it works
Let users merge/split clusters. Learn from their corrections over time.

```
Cluster: "Hover Issues" (15 items)
[✓ Looks good]  [Split this cluster]  [Merge with another]

User action: Merges "Hover Issues" with "Tooltip Bugs"
System learns: For this repo, threshold was too tight
```

### Implementation
- Store user corrections (merge/split actions)
- Track which pairs of issues users group together
- Use corrections to adjust future thresholds
- Could use simple averaging or ML model

### Pros
- Gets better with use
- Personalized per user/repo
- Captures domain knowledge

### Cons
- Most complex to build
- Requires user engagement
- Cold start problem (no learning initially)

---

## Recommendation

**Start with: Auto-tuning + User Slider**

1. **Auto-tuning** as default - picks good threshold automatically
2. **User slider** as override - power users can adjust if needed

This gives:
- Zero-config for most users (auto-tuning handles it)
- Escape hatch for edge cases (slider)
- Low implementation complexity

**Future enhancement:** Add hierarchical view for exploration use cases.

---

## Next Steps

1. [ ] Decide which approach(es) to implement
2. [ ] Design UI changes needed
3. [ ] Implement in notebook first (prototype)
4. [ ] Integrate into Soulcaster backend
