# Project Title: Fetal Sex-Specific Drug Repurposing for Late-Onset Preeclampsia

## Project Goal

The primary goal of this project is to use transcriptomics data from placental tissue to identify a sex-specific disease signature in late-onset preeclampsia (PE) and use a large-scale drug perturbation database (TAHOE-100M) to propose existing, safe-for-pregnancy drugs for repurposing.

**Hypothesis:** The gene expression changes observed in male-bearing placentas with preeclampsia can be reversed by FDA-approved drugs that are generally considered safe during pregnancy, suggesting a novel therapeutic pathway.

## Scientific Rationale (The "Why")

The original study (GSE306864) found that late-onset Preeclampsia only causes significant transcriptional changes in **male-bearing placentas**. Female-bearing placentas appear resilient, showing no significant differential gene expression.

Here, we will focus solely on the differential signature between **Male Preeclampsia** and **Male Control** placentas. This signature represents the core transcriptional difference that we want to *reverse* with a therapeutic drug.

## Resources Provided

1. **Raw Count Matrix:** A file containing gene-level counts for all samples (rows=genes, columns=samples).
2. **Metadata File:** A file containing sample-to-phenotype mapping (e.g., Sample ID, Fetal Sex, Preeclampsia Status). *You must first clean this file using the provided Python script to ensure proper labeling.*
3. **TAHOE-100M Exploration Script:** A Python script utilizing the Hugging Face API to query 100,000 random cells and understand their drug perturbation data.

## Step-by-Step Guide and Analytical Approach

### Phase 1: Data Preparation and Differential Expression (DE)

**Objective:** Isolate the disease-specific gene signature in male-bearing placentas.

| Task | Deliverable | Key Analytical Thought |
| --- | --- | --- |
| **Metadata Cleaning** | Cleaned `metadata.csv` file. | The GEO metadata is messy. Use the provided Python script to create a single, clean table with columns: `Sample ID`, `Sex`, and `Condition` (PE/Control). |
| **Subset Data** | `Male_Subset_Metadata.csv` | **Filter the metadata to include only male-bearing samples.** This is critical to isolate the sex-specific effect. |
| **Prepare Count Data** | Matched Count Matrix. | Filter the main Raw Count Matrix to only include the columns (samples) identified in the `Male_Subset_Metadata.csv` file. **Ensure sample IDs match perfectly.** |
| **Differential Expression** | `DE_Results.csv` (Upregulated Genes) | Perform a differential expression analysis (e.g., using R/Bioconductor with **DESeq2** or Python with **PyDESeq2**) comparing **Male PE** vs. **Male Control**. |
| **Select Signature** | `Preeclampsia_Signature.txt` | Filter the DE results: Select all genes that are **Significantly Upregulated** in Male PE (e.g., Log2FoldChange > 0 and adjusted p-value < 0.05). **This is your "Bad" Signature.** |

### Phase 2: Drug Repurposing (The Search)

**Objective:** Find drug perturbations that *reverse* the "Bad" Signature.

| Step | Task | Deliverable | Key Analytical Thought |
| --- | --- | --- | --- |
| **2.1** | **Query TAHOE-100M** | Initial list of drug hits. | Use the provided TAHOE-100M script. Input your `Preeclampsia_Signature.txt` and look for drugs that induce an **Inverse Signature**. That is, drugs that **downregulate** the genes you found to be upregulated in PE. |
| **2.2** | **Filter Cell Lines** | Refined drug list. | **Crucial Step:** The placental DE signature is heavily influenced by immune/endothelial cells. When querying TAHOE, prioritize hits derived from non-cancerous **Immune Cell Lines** (e.g., macrophages, T-cells) or **Endothelial Cell Lines** (if available) over general cancer lines. If not possible, keep all hits for now. |
| **2.3** | **Score Reversal** | Top 50 Drug Candidates. | Rank the drug hits based on the similarity of the induced signature to the *perfect inverse* of your PE signature. Prioritize the top 50 drugs with the best **Reversal Score**. |

### Phase 3: Clinical Filtering (The Safety Check)

**Objective:** Filter the top drug hits for clinical relevance and safety in pregnancy.

| Step | Task | Deliverable | Key Analytical Thought |
| --- | --- | --- | --- |
| **3.1** | **Toxicity Check** | Drug Safety List. | For the Top 50 drugs, look up each one's primary mechanism of action. **IMMEDIATELY DISCARD** any drugs that are cytotoxic, anti-proliferative, or known to interfere with basic cell growth (most traditional cancer chemotherapies). |
| **3.2** | **Pregnancy Safety** | Final Shortlist (Top 5-10). | Use resources like the FDA's Pregnancy and Lactation Labeling Rule (PLLR) or reputable public databases (e.g., Reprotox, Teratogen Information Services) to determine the pregnancy risk category (A, B, C, D, or X). **Focus on Categories A, B, or low-risk C.** |
| **3.3** | **Literature Review** | Drug Summary Table. | For the final shortlist of 5-10 safe candidates, conduct a brief literature search. Have any of these drugs been studied in the context of inflammation, endothelial dysfunction, or preeclampsia previously? |
