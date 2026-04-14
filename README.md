# Spl-IsoFind Reproducibility

This is GitHub for reproducing analysis from

[Michielsen, L., Prjibelski, A.D., Foord, C., Hu, W., Jarroux, J., Hsu, J., Tomescu, A.I., Hajirasouliha, I. and Tilgner, H.U., 2025. Spatial isoform sequencing at sub-micrometer single-cell resolution reveals novel patterns of spatial isoform variability in brain cell types. bioRxiv.
](https://www.biorxiv.org/lookup/doi/10.1101/2025.06.25.661563)

This repository contains 3:
1. The long-read analysis pipeline
2. Figure reproducibility
3. Data simulation and algorithm quality assessment

## Long-read analysis pipeline

### 1. Running Spl-IsoQuant-2

All long-read datasets were analysed with [Spl-IsoQuant](https://github.com/algbio/spl-IsoQuant).
The resulting files, such as extended annotations and TSV files with all 
read information (allinfo) are available on Zenondo.

See command lines [here](SplIsoQuant.md).

### 2. Preprocessing the allinfo files
Input (download from Zenodo?)
- Allinfo files
- Short-read adata object with metadata (celltypes, brain regions etc)
- CIDmap (map from barcode to cellID)

Code: add 

Output: filtered and annotated allinfo file

### 3. Scisorseqr analysis

### 4. Spl-IsoFind analysis

## Figure reproducibility
Add notebooks to reproduce figures


## Data simulation and algorithm quality assessment

