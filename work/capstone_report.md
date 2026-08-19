# Capstone Report — CTR & Engagement Opportunity Scoring

**Author:** Clarisse Jem Salazar
**Lane:** Lane 4 (CTR / Engagement Opportunity Scoring)
**Repo:** https://github.com/jemslzr/flyrank-ml
**Date:** August 19, 2026

---

### 0. Abstract
This project investigates which highly visible pages in the FlyRank ecosystem systematically under-capture clicks relative to their organic search ranking. Using a 30-day historical snapshot from the `engagement_fix` dataset, I trained a Random Forest Regressor to predict a page's expected click-through rate (CTR) based on its exact ranking position and impression volume. The model successfully outperformed a standard position-tier baseline, reducing Mean Absolute Error by capturing the non-linear decay of search visibility. The final output is an automated, ranked action playbook that directs content editors to the most profitable title and meta description optimizations.

### 1. Problem framing
This analysis supports the content team's prioritization workflow. The unit of analysis is a single page over a 30-day window. The output is a ranked queue scored by "expected missed clicks." A human editor acts on this by reviewing and rewriting the metadata for the top-flagged pages. A false positive costs an editor 15 minutes of wasted effort, while a false negative leaves easy traffic uncaptured. Machine learning improves this process by accounting for the continuous, non-linear interactions between search position and impression volume that fixed "if/then" rules miss.

### 2. Data safety
I used the `engagement_fix` subset of the FlyRank warehouse release on Hugging Face. The analysis utilizes a mid-panel 30-day snapshot to avoid temporal leakage. Features include `impressions_30d` and `avg_position_30d`. I explicitly excluded `clicks_30d` from the feature set to prevent perfect target leakage (since CTR is mathematically derived from clicks). To ensure public safety, all `content_hash_id` and `client_hash_id` strings remain pseudonymized, and no client domains, raw URLs, or private search queries are exposed.

### 3. Baseline
The baseline was a transparent rule that grouped pages into four position tiers (Top 3, Page 1 Bottom, Page 2, Deep) and calculated the median CTR for each tier. Pages were then scored by the gap between their actual CTR and their tier's median. This is a fair comparison because it represents the standard spreadsheet pivot-table methodology most SEO teams currently use to prioritize work.

### 4. Model / analysis
I framed this as a regression task, utilizing a Random Forest Regressor. Tree-based models are ideal for this lane because they naturally handle the heavy-tailed distribution of search impressions and the exponential decay of CTR by position without requiring complex mathematical transformations. The target proxy definition is the historical `ctr_30d` for the given page.

### 5. Evaluation
I used a `GroupShuffleSplit` grouped by `client_hash_id` for validation. This ensures that pages from the same website do not appear in both the training and test sets, preventing the model from simply memorizing a specific client's overall brand performance. Evaluated on this honest split, the Random Forest model achieved a lower Mean Absolute Error (MAE) than the fixed tier-median baseline, proving it learned generalized, transferable search signals.

### 6. Interpretation
Feature importance extraction confirmed that `avg_position_30d` is the dominant driver of expected CTR, but `impressions_30d` serves as a critical confidence weight. Error analysis showed the model occasionally underpredicts CTR for navigational brand queries (where users almost always click the first link) and overpredicts CTR for informational queries that Google answers directly via Zero-Click Featured Snippets. 

### 7. Recommendation
The model's output powers an automated action playbook. Editors should start at the top of the ranked queue and manually review the live SERP intent before rewriting metadata. The model flags directional opportunities; it cannot guarantee that a rewrite will cause a traffic recovery. Pages with under 1,000 monthly impressions are strictly excluded from the queue to prevent noisy, low-value optimizations.

### 8. Reproducibility
The full pipeline is available in the linked repository under `work/notebooks/capstone.ipynb`. 
*   **Environment:** Run `pip install -r requirements.txt`.
*   **Execution:** Run all cells in `capstone.ipynb`. 
*   **Randomization:** All splits and model initializations are locked to `random_state=42`.

### 9. Acknowledgments & data credit
Built on the FlyRank ML Internship dataset. Data provided by [FlyRank](https://flyrank.ai).
