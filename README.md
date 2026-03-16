# Retrieval-Augmented Regression for Sequence Similarity Estimation 

This project explores retrieval-augmented models for DNA sequence similarity prediction, drawing inspiration from classical bioinformatics tools such as BLAST. In BLAST, a query sequence is compared against a database to find highly similar sequences, and similarity scores are computed based on alignments. Analogously, this project implements a RAG-style neural network model that combines learned embeddings with database retrieval to predict the similarity between DNA sequences.

## Main Idea
The key concept is that RAG models naturally mimic BLAST’s workflow:

1. **query embedding** (a DNA sequence is transformed into a high-dimensional vector that captures local motifs and patterns);
2. **database retrieval** (the most relevant sequences are retrieved from a precomputed reference database, analogous to BLAST’s search step);
3. **attention-based fusion** (the model attends to retrieved sequences, integrating context into the query embedding);
4. **similarity prediction** (a regression head predicts the similarity score between sequences);

This approach allows the model to leverage relational patterns across sequences, not just the intrinsic features of the query, thereby improving predictive accuracy in tasks where local context matters. 

## Dataset
1. 16S rRNA sequences were  filtered for valid nucleotides (A, T, C, G), and limited to 200 sequences across 50 target bacterial species;
2. pairwise similarity labels (HSP identities) were computed using BLAST for all sequence pairs;
3. for the RAG dataset, each query pair is augmented with the most similar sequence pair from the database (excluding the query itself), forming a retrieval-augmented training set. 

## Workflow Overview

1. **Data Processing:**
    - DNA sequences are one-hot encoded;
    - sequence pairs include both the query and retrieved reference sequences.
    - data is split into training, validation, and test sets.

2. **Architecture:**
    - DNANoRAG is a baseline model that encodes DNA sequences using a convolutional SeqEncoder and predicts similarity with a simple MLP, relying solely on the query sequence features;
    - DNARAG extends this by retrieving top-K similar sequences from a database, applying cross-attention between the query and retrieved embeddings, and combining them in an MLP to predict similarity. This retrieval-augmented approach allows the model to leverage relational context, similar to how BLAST compares sequences against a database.

3. **Training:**
    - models are trained using huber loss;
    - RAG model can refresh database embeddings at intervals to ensure up-to-date retrieval;
    - training tracks both train and validation losses across epochs.

4. **Evaluation Metrics:**
  - regression metrics (MAE, RMSE, R², MRE);
  - correlation metrics (Pearson and Spearman);
  - RAG-specific metric (top-K cosine similarity between query and retrieved embeddings);
  - visualizations include residual plots and predicted vs. true similarity scatterplots.

## Results and Analysis
Both models converged rapidly, with training and validation losses dropping to near-zero by epoch 7. The baseline model outperformed the RAG model on direct regression metrics (MAE: 0.0104 vs. 0.0162, R²: 0.9299 vs. 0.8604), likely because it directly maps input sequences to similarity scores without relying on retrieved context. This allows it to fine-tune embeddings specifically for the regression objective, capturing subtle nucleotide-level patterns.

The RAG model, on the other hand, achieved near-perfect retrieval of similar sequences from the database (top-K Cosine: 0.9998), demonstrating that it effectively learns relational information similar to BLAST alignments. Its slightly lower regression accuracy suggests that while retrieval provides rich contextual information, integrating this information into precise numeric similarity predictions introduces additional variance.

In short, the baseline excels at precise regression for seen patterns, while RAG stands out in relational reasoning and retrieval-based generalization, reflecting a trade-off between direct prediction fidelity and contextual awareness. 



