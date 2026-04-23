# Running the spatial downstream analysis
Spl-IsoFind can be downloaded from the [official repository](https://github.com/tilgnerlab/Spl-IsoFind).
[Version 0.1.8](https://github.com/tilgnerlab/Spl-IsoFind/releases/) was used for the analysis. 

scisorseqr was used for comparisons of predefined regions. scisorseqr can be downloaded [here](https://github.com/tilgnerlab/scisorseqr). 

All input, intermediate, and output files mentioned in the table and/or code snippets can be downloaded [here](https://doi.org/10.5281/zenodo.19499423).

## Preprocessing the allinfo files (Stereo-seq and Visium HD)
During preprocessing, unspliced reads and reads that do not overlap the tissue section are filtered out, and region and cell-type labels are assigned to every read. 

``` python
import SplIsoFind
SplIsoFind.pp.allinfo_addct(allinfo,
                            bc2cidmap,
                            adata,
                            allinfo_filt)
```

Three files are needed during preprocessing:
- allinfo: the output from Spl-IsoQuant
- bc2cidmap: a barcode to cellID mapping
- adata: an AnnData object with metadata including celltype and region assignments

The preprocessing results are saved to a filtered and labeled allinfo file (allinfo_filt).

The correspondence between the data and the input files is provided in the table below.

| Sample name     | allinfo                                        | bc2cidmap                          | adata                         | allinfo_filt    |
|-----------------|------------------------------------------------|------------------------------------|-------------------------------|-------------|
| S1 (AE) ONT     | S1_ONT.UMI_filtered.ED4.allinfo.gz             | Sample1_barcodeToPos.CellID.tsv.gz | Sample1_cellbin_adjusted.h5ad | S1_ONT.UMI_filtered.ED4.allinfo_filtered.gz     |
| S1 (3.3K) ONT   | S1_4kGenesJunctions.UMI_filtered.ED4.allinfo.gz| Sample1_barcodeToPos.CellID.tsv.gz | Sample1_cellbin_adjusted.h5ad | S1_4kGenesJunctions.UMI_filtered.ED4.allinfo_filtered.gz       |
| S1 (AE) PB      | S1_PB.UMI_filtered.ED4.allinfo.gz              | Sample1_barcodeToPos.CellID.tsv.gz | Sample1_cellbin_adjusted.h5ad | S1_PB.UMI_filtered.ED4.allinfo_filtered.gz     |
| S2 (AE) ONT     | S2_ONT.UMI_filtered.ED4.allinfo.gz             | Sample2_barcodeToPos.CellID.tsv.gz | Sample2_cellbin_adjusted.h5ad | S2_ONT.UMI_filtered.ED4.allinfo_filtered.gz    |
| S2 (AE) PB      | S2_PB.UMI_filtered.ED4.allinfo.gz              | Sample2_barcodeToPos.CellID.tsv.gz | Sample2_cellbin_adjusted.h5ad | S2_PB.UMI_filtered.ED4.allinfo_filtered.gz    |
| V1 ONT          | V1_ONT_2um.UMI_filtered.ED4.allinfo.gz         | V1_barcode.CellID.tsv.gz           | V1_cell.h5ad                  | V1_ONT_2um.UMI_filtered.ED4.allinfo.filtered.labeled.gz     |
| V1 PB           | V1_PB_2um.UMI_filtered.ED4.allinfo.gz          | V1_barcode.CellID.tsv.gz           | V1_cell.h5ad                  | V1_PB_2um.UMI_filtered.ED4.allinfo.filtered.labeled.gz     |
| V2 ONT          | V2_ONT_8um.UMI_filtered.ED4.allinfo.gz         | V2_barcode.CellID.tsv.gz           | V2_8um.h5ad                   | V2_ONT_8um.UMI_filtered.ED4.allinfo.filtered.labeled.gz    |
| V3 PB           | V3_PB_8um.UMI_filtered.ED4.allinfo.gz          | V3_barcode.CellID.tsv.gz           | V3_8um.h5ad                   | V3_PB_8um.UMI_filtered.ED4.allinfo.filtered.labeled.gz    |

## Preprocessing the allinfo files (Visium)
Preprocessing for the Lebrigand data is slightly easier because spots do not need to be grouped. This Python snippet only uses the allinfo file.

| Sample name     | fn_allinfo                                        | 
|-----------------|---------------------------------------------------|
| CBS1            | Spatial.Lebrigand.CBS2.UMI_filtered.ED3.allinfo.gz| 
| CBS2            | Spatial.Lebrigand.CBS2.UMI_filtered.ED3.allinfo.gz| 

``` python
import pandas as pd
allinfo = pd.read_csv(fn_allinfo, sep='\t', index_col=0, header=None)

# Filter for reads without region label 
allinfo = allinfo[allinfo[2].notna()]
allinfo = allinfo[allinfo[2] != 'Outside']

# Filter for spliced reads
allinfo = allinfo[allinfo[10] > 0]

# Remove whitespace in region labels
allinfo[2] = allinfo[2].str.replace(' ', '_', regex=False)

allinfo.to_csv(f'{fn_allinfo}.filtered.labeled.gz',
            header=False, sep='\t', index=True, compression='gzip')
```


## Running Spl-IsoFind

### Step 1: Create the isoform matrix (Stereo-seq and Visium HD).
This matrix stores the relative expression counts (rows are cells, columns are isoforms).

Input: 
- allinfo_filt: the filtered allinfo file previously created
- bc2cidmap: a barcode to cellID mapping
- adata: an AnnData object with metadata including celltype and region assignments

The function saves three files to the output directory (isodir):
- X_sparse.npz: CSR matrix of relative expression values (rows = cells, columns = isoforms).
- genes_isoforms.csv: List of (Gene, Isoform) pairs.
- labels.csv: Cell metadata from AnnData.

``` python
import SplIsoFind

# When creating an isoform matrix for 1 slide
SplIsoFind.pp.create_isoform_matrix(allinfo_filt,
                                    bc2cidmap,
                                    adata,
                                    isodir)

# When combining two consecutive slides in the analysis (e.g. S1 (AE) ONT and S2 (AE) ONT)
SplIsoFind.pp.create_isoform_matrix_twoslides(allinfo_filt1,
                                              allinfo_filt2,
                                              bc2cidmap1,
                                              bc2cidmap2,
                                              adata1,
                                              adata2,
                                              isodir)

```
### Step 1: Create the isoform matrix (Visium).
Again, this step is slightly different for the Lebrigand data because spots do not need to be grouped.

| Sample name     | labels                                            | 
|-----------------|---------------------------------------------------|
| CBS1            | CBS1/labels.csv                                   | 
| CBS2            | CBS2/labels.csv                                   | 


``` python
import SplIsoFind

# When creating an isoform matrix for 1 slide
SplIsoFind.pp.create_isoform_matrix_spot(allinfo_filt,
                                         labels,
                                         isodir)
```


### Step 2: Run Moran's I
Load the isoform matrix previously created. When running Moran's I on two consecutive slides (e.g. S1 (AE) ONT and S2 (AE) ONT), the aligned x and y coordinates should be used.

``` python
import SplIsoFind

# Load the created isoform matrix
x_sparse, labels, isoforms = SplIsoFind.pp.load_sparse(iso_dir)

# Run Moran's I
mI, pval, qval = SplIsoFind.sv.moransI_sparse(x_sparse,
                                              labels,
                                              isoforms,
                                              k = 50,
                                              mincells = 250)

# Run Moran's I on aligned slides
mI, pval, qval = SplIsoFind.sv.moransI_sparse(x_sparse,
                                              labels,
                                              isoforms,
                                              x = 'x_aligned',
                                              y = 'y_aligned',
                                              k = 50,
                                              mincells = 250)


```

