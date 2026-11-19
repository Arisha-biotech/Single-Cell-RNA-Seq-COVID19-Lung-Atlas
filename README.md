# Single-Cell-RNA-Seq-COVID19-Lung-Atlas
#### Single-Cell RNA-Seq Analysis of Human Lung Tissue in COVID-19 (GSE171524)

#### Overview

In this project, I analyzed single-cell RNA sequencing (scRNA-Seq) data from human lung tissue to see how severe COVID-19 changes lung cells at the individual cell level. Using Python, Scanpy, and deep-learning batch integration (scvi-tools), I processed **27 patient lung samples** (20 COVID-19 patients and 7 healthy controls) containing **113,972 clean cells**. 

My main goal was to clean noisy single-cell data, align samples from different patients without crashing memory limits, identify major lung cell types, and compare how cell populations shift during COVID-19 infection. 

#### Dataset Used

* **Source:** NCBI Gene Expression Omnibus (GEO Accession: **GSE171524**)
* **Samples:** 27 human lung tissue samples (78,031 COVID-19 cells and 35,941 Healthy Control cells after quality control)
* **Tools & Libraries:** Python, Scanpy, AnnData, scvi-tools (SCVI, SOLO), SciPy (csr_matrix), Pandas, NumPy, Matplotlib, and leidenalg

#### Step-by-Step Workflow

#### 1. Data Download & Single-Sample Exploration

* Downloaded the raw dataset (GSE171524_RAW.tar) directly from the NCBI FTP server and extracted all 27 compressed count matrices.
* Tested the workflow on a single healthy control sample (GSM5226574_C51ctr, 6,099 cells × 34,546 genes).
* Trained SCVI and SOLO deep-learning models to detect and remove doublets (two cells accidentally captured together).
* Filtered out low-quality and dying cells by removing cells with high mitochondrial genes (< 20%), ribosomal genes (< 2%), and extreme gene count outliers (top 98th percentile), leaving **5,977 clean cells**.

#### 2. Memory-Safe Integration of All 27 Samples

* Combining 113,972 cells at once can easily crash standard cloud RAM limits (12.7 GB).
* To keep memory usage low, I wrote a preprocessing function (pp()) that converts every count matrix into a compressed sparse format (scipy.sparse.csr_matrix) and frees unused RAM (gc.collect()) after loading each patient file.
* Normalized total counts per cell (target_sum=1e4), applied log transformation (log1p), and selected the **2,000 most highly variable genes** across batches.

#### 3. Batch Correction & Clustering

* Trained an SCVI model across all 27 samples while accounting for patient-to-patient variation (Sample) and technical noise (pct_counts_mt, total_counts, pct_counts_ribo).
* Built a nearest-neighbor graph using scikit-learn (transformer="sklearn") on the batch-corrected latent space (X_scVI), generated UMAP embeddings, and grouped cells into clusters using the Leiden algorithm (resolution=0.5).

#### 4. Marker Gene Discovery & Cell Type Annotation

* Ran the Wilcoxon rank-sum test (rank_genes_groups) to find the top marker genes for every cluster.
* Scored each cluster against known lung cell marker genes (score_genes) to label major cell populations, including **AT1 Cells, AT2 Cells, Macrophages, Monocytes, T Cells, NK Cells, B & Plasma Cells, Ciliated Epithelial Cells, Fibroblasts, Endothelial Cells, and Smooth Muscle Cells**.

#### Key Results & Visuals

#### 1. Annotated Lung Cell Atlas (UMAP)

The left plot shows the 10 major lung cell populations identified across the integrated cohort, and the right plot shows how cells from COVID-19 patients and healthy controls mix cleanly after batch correction. 

 

#### 2. Marker Gene Verification (Dot Plot)

This dot plot confirms that each labeled cell type strongly expresses its expected biological markers (for example, SFTPC and SFTPA1 in AT2 cells, AGER in AT1 cells, MARCO and CD68 in Macrophages, and CD3D in T Cells). 

 

#### 3. Cellular Remodeling in COVID-19 vs. Healthy Lungs

Comparing cell percentages between conditions highlights major tissue remodeling during severe infection: 

* **Loss of Healthy Alveolar Cells:** Healthy control lungs have a much higher proportion of alveolar epithelial cells (**AT1** and **AT2** cells) needed for normal gas exchange.
* **Immune & Fibrotic Expansion:** COVID-19 lungs show a large expansion of **Macrophages**, **T Cells**, and **Fibroblasts**, reflecting strong inflammation and early tissue scarring (fibrosis).

 

#### Repository Files

* COVID19_Lung_scRNA_Seq_Atlas.ipynb — Complete Jupyter/Colab notebook with all code cells, outputs, and charts.
* umap_annotated_covid19_lung_atlas.png — Integrated UMAP colored by annotated cell type and disease condition.
* dotplot__lung_celltype_markers.png — Dot plot of canonical marker genes across lung cell types.
* condition_cell_proportions.png — Stacked bar chart comparing cell percentages in COVID-19 vs. Control lungs.
* top_100_cluster_markers.csv — Top 100 differentially expressed marker genes per cluster from Wilcoxon rank-sum testing.
* README.md — Project overview, methods, and biological summary.

#### How to Run

1. Open COVID19_Lung_scRNA_Seq_Atlas.ipynb in **Google Colab** (select a **T4 GPU** runtime for fast scVI model training).
2. Run the cells in order from top to bottom—the first cell downloads the dataset directly from NCBI GEO.
