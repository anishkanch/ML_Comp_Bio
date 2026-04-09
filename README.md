# MLB S26 Hackathon - ML-Guided Protein Engineering
 
---
 
## Overview
 
This project uses machine learning to predict the fitness (DMS score) of single-site protein mutations and identify the top 10 mutations most likely to improve protein function. We use an active learning pipeline with an ensemble of 5 neural networks and three query rounds to iteratively improve our model.
 
---
 
## Requirements
 
Run the following in your environment before starting:
 
```
pip install torch numpy pandas scipy scikit-learn
```
 
The notebook was developed and tested on Google Colab.
 
---
 
## Files Required
 
Before running the notebook, upload these three files to the Colab runtime (or your working directory):
 
| File | Description |
|---|---|
| `sequence.fasta` | Wild-type protein sequence (656 amino acids) |
| `train.csv` | Initial training data with mutant names and DMS scores |
| `test.csv` | Test set of 11,324 mutants to predict |
 
After receiving query results from the autograder, also upload:
 
| File | Description |
|---|---|
| `query1_results.csv` | DMS scores returned after Query 1 |
| `query2_results.csv` | DMS scores returned after Query 2 |
| `query3_results.csv` | DMS scores returned after Query 3 |
 
---
 
## How to Run
 
Run each cell in the notebook in order from top to bottom. Do not skip cells. The notebook is organized into the following sections:
 
### Section 1: Load Data
Loads the wild-type sequence, training data, and test data. Adds the mutated sequence string to each dataframe.
 
### Section 2: Encoding and Model Definition
 
**Encoding:** Each mutation is encoded as a 21-dimensional vector:
- Position 0: normalized position in sequence (position / 656)
- Positions 1-20: one-hot encoding of the new amino acid
 
**Model (ProteinNet):** A 3-layer MLP:
```
Input (21) → Linear(128) → ReLU → Dropout(0.2)
           → Linear(64)  → ReLU → Dropout(0.2)
           → Linear(1)   → Sigmoid → Output
```
 
### Section 3: Train 5 Models (Ensemble)
 
Five identical ProteinNet models are trained independently, each with a different random seed (0 through 4). Each model:
- Is trained on the same training data
- Uses an 80/20 train/validation split
- Runs for 1000 epochs with Adam optimizer (lr=0.001) and MSE loss
- Prints validation Spearman correlation every epoch
 
All 5 trained models are stored in a list called `ensemble`.
 
After all 5 models are trained, each makes predictions on all 11,324 test mutants. The mean and standard deviation of predictions across the 5 models represent the predicted DMS score and uncertainty for each mutant respectively.
 
### Section 4: Query 1 — Thompson Sampling
 
Thompson Sampling is used to select 100 mutants for the first query. For each of the 100 query slots, one of the 5 models is randomly chosen and its top-ranked mutant (not yet selected) is picked. This naturally balances exploring uncertain regions with exploiting high-predicted-score mutants.
 
Output: `query.txt` — submit this to the Gradescope Hackathon-ActiveLearning assignment.
 
### Section 5: Retrain After Query 1
 
After receiving `query1_results.csv` from the autograder, add those 100 labeled mutants to the training data and retrain all 5 models from scratch on the expanded dataset.
 
### Section 6: Query 2 — Upper Confidence Bound (UCB)
 
UCB selects the next 100 mutants using:
 
```
UCB score = mean prediction + 4 × standard deviation
```
 
The weight of 4 balances exploration (uncertainty) and exploitation (predicted fitness). Mutants already queried in Query 1 are excluded.
 
Output: `query.txt` — submit to the Gradescope Hackathon-ActiveLearning assignment.
 
### Section 7: Retrain After Query 2
 
After receiving `query2_results.csv`, add those 100 labeled mutants to training data and retrain all 5 models again.
 
### Section 8: Query 3 — Greedy
 
By this point the model has seen enough data to be trusted more directly. Greedy simply picks the 100 test mutants with the highest mean predicted DMS scores. Mutants from Query 1 and Query 2 are excluded.
 
A sanity check cell verifies that none of the selected mutants appear in the original training set.
 
Output: `query.txt` — submit to the Gradescope Hackathon-ActiveLearning assignment.
 
### Section 9: Final Retrain and Submission
 
After receiving `query3_results.csv`, add those 100 labeled mutants to training data and retrain all 5 models one final time. Then generate the two required submission files:
 
**predictions.csv** — mean predicted DMS score for all 11,324 test mutants:
```
mutant, DMS_score_predicted
V1D, 0.732
...
```
 
**top10.txt** — the 10 test mutants with the highest mean predicted scores, one per line:
```
A649F
E651F
...
```
 
---
 
## Output Files
 
| File | Where to Submit | Description |
|---|---|---|
| `predictions.csv` | Gradescope Leaderboard | Predicted DMS scores for all test mutants |
| `top10.txt` | Gradescope Leaderboard | Top 10 recommended mutations |
| `query.txt` | Gradescope Hackathon-ActiveLearning | 100 mutants to query (generated 3 times) |
 
---
 
## Active Learning Pipeline Summary
 
```
Original training data (1,140 examples)
        ↓
Train ensemble of 5 MLPs
        ↓
Query 1: Thompson Sampling → 100 new labeled mutants
        ↓
Retrain ensemble (1,240 examples)
        ↓
Query 2: UCB → 100 new labeled mutants
        ↓
Retrain ensemble (1,340 examples)
        ↓
Query 3: Greedy → 100 new labeled mutants
        ↓
Final retrain (1,440 examples)
        ↓
Generate predictions.csv and top10.txt
```
 
---
 
## Team Contributions
 
| Member | Contribution |
|---|---|
| Greyson | Model 1 training, Thompson Sampling query cell |
| Vishaal | Models 2-5 training, retrain cells between queries |
| Anish | UCB query cell |
| Yash | Greedy query cell, final predictions and top10 generation |
