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
 
Run each cell in the notebook in order from top to bottom. Do not skip cells.
 
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
