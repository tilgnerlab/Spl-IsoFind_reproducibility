# Running Spl-IsoQuant

Spl-IsoQuant can be downloaded from the [official repository](https://github.com/algbio/spl-IsoQuant).
[Version 2.4](https://github.com/algbio/spl-IsoQuant/releases/tag/v2.4.0) was used for the analysis.

## Stereo-seq data

Stereo-seq data used in this paper is available on [SRA](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA1282707).
Barcode whitelists can be downloaded [here](https://doi.org/10.5281/zenodo.19499423).

The correspondence between the data and barcode whitelist is provided in the table below.

| Sample name     | Slide | Technology | SRA number               | Barcodes    |
|-----------------|-------|------------|--------------------------|-------------|
| Sample 1 (AE)   | S1    | ONT        | SRR34231613, SRR34231614 | B04571D     |
| Sample 1 (3.3K) | S1    | ONT        | SRR34231612              | B04571D     |
| Sample 1 (AE)   | S1    | PacBio     | SRR34231608              | B04571D     |
| Sample 2 (AE)   | S2    | ONT        | SRR34231611              | D04620C2    |
| Sample 2 (AE)   | S2    | PacBio     | SRR34231607              | D04620C2    |



#### Transcript discovery (all data combined)

``` 
splisoquant.py -d ont --mode bulk \
--reference GRCm39.primary_assembly.genome.fa \
--genedb gencode.vM36.basic.annotation.gtf --complete_genedb \
--fastq <FASTQ files>
-p Stereo.ALL -o <output folder> 
```

#### Quantification (individual processing)

The extended GTF was obtained using the previous set, but can be also downloaded [here](https://doi.org/10.5281/zenodo.19499423).

``` 
splisoquant.py -d <pacbio|ont> --mode stereoseq \
--reference GRCm39.primary_assembly.genome.fa \
--genedb Stereo.ALL.extended_annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
--barcode_whitelist <Barcodes.tsv> \
--no_model_construction --large_output allinfo \ 
-p Stereo -o <output folder>
```
 

## 10x Visium HD data

10x Visium HD data used in this paper is available on [SRA](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA1282707) and at public repositories.
Barcode whitelists and supplementary files are proprietary and are available on request.

| Sample name   | Technology | SRA number / Link                                                                                                                 |
|---------------|------------|-----------------------------------------------------------------------------------------------------------------------------------|
| Visium HD V1  | ONT        | SRR37732230                                                                                                                       |
| Visium HD V1  | PacBio     | SRR37732229                                                                                                                       |
| Visium HD V2  | ONT        | [Link](https://epi2me.nanoporetech.com/visium_hd_2025.06/)                                                                        |
| Visium HD V3  | PacBio     | [Link](https://downloads.pacbcloud.com/public/dataset/Kinnex-single-cell-RNA/DATA-RevioSPRQ-Kinnex-VisiumHD-mouseBrain/1-Sreads/) |


#### Barcode detection 

Detecting barcodes sequences:
``` 
splisoquant_detect_barcodes.py --mode visium_hd \ 
--barcpdes <Barcodes1.tsv> <Barcodes2.tsv> \
--fastq <FASTQ files> -o <output prefix> 
```

#### Quantification

Converting barcode sequences to spot ids with different resolution (2um, 8um, 16um):
```
misc/barcodes_seq_to_spot_id.py <barcoded_reads.tsv> \
<visium_hd.barcode1_spots.tsv> <visium_hd.barcode2_spots.tsv> <spot_mappings.tsv> \  
-o <Read2Spot.tsv> 
```
The extended GTF was obtained using the Stereo-seq data and can be downloaded here.

``` 
splisoquant.py -d <pacbio|ont> --mode visium_hd \
--reference GRCm39.primary_assembly.genome.fa \
--genedb Stereo.ALL.extended_annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
--barcoded_reads <Read2Spot.tsv> \
--no_model_construction --large_output allinfo \
-p VisiumHD -o <output folder> 
```

Alternatively, barcode-to-spot conversion can be done on the fly to skip barcode conversion step:

``` 
splisoquant.py -d <pacbio|ont> --mode visium_hd \
--reference GRCm39.primary_assembly.genome.fa \
--genedb Stereo.ALL.extended_annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
--barcoded_reads <barcoded_reads.tsv> \
--barcode2barcode <barcode2spots.tsv:0:X> \
--no_model_construction --large_output allinfo \
-p VisiumHD -o <output folder> 
```

 
## 10x Visium data from Lebrigand et al.

We used the following ONT data from [Lebrigand et al., 2023](https://academic.oup.com/nar/article/51/8/e47/7079641):

- CBS1:
  - SRR12157792
  - SRR12157793
  - SRR13938447
- CBS2:
  - SRR12157785
  - SRR12157786
  - SRR12157787
  - SRR12157788
  - SRR12157789
  - SRR12157790
  - SRR12157791
  - SRR13938446

The data is available on [SRA](https://www.ncbi.nlm.nih.gov/bioproject/PRJNA644362).
Barcodes and their corresponding cell-types were extracted for RDS files, which are [available on GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE153859)
or at [our Zenodo repository](https://doi.org/10.5281/zenodo.19499423).


#### Transcript discovery (CBS1 + CBS2)

``` 
splisoquant.py -d ont --mode bulk \
--reference GRCm39.primary_assembly.genome.fa \
--genedb gencode.vM36.basic.annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
 -p Spatial.Lebrigand --large_output none -o <output folder> 
```

#### Quantification (CBS1 and CBS2 separately)

The extended GTF was obtained using the previous set, but can be also downloaded [here](https://doi.org/10.5281/zenodo.19499423).

```
splisoquant.py  -d ont --mode visium_5prime \ 
 --reference GRCm39.primary_assembly.genome.fa \
 --genedb Spatial.Lebrigand.extended_annotation.gtf --complete_genedb \
 --bam <FASTQ files> \
 --barcode_whitelist <Barcodes.tsv> \
 --barcode2spot <Barcode2Cluster.tsv>
 --no_model_construction --large_output allinfo \
 -p Spatial.Lebrigand -o <output folder>
```

## 10x single-cell human data from Joglekar et al.

We used the dataset from [Joglekar et al., 2024](https://www.nature.com/articles/s41593-024-01616-4)
that contains six human brain samples (3 females, 3 males) sequenced with 10x single-cell via ONT 
(data available [here](https://knowledge.brain-map.org/data/ASP3B09DZ8PXDUYSHDH)).

#### Transcript discovery (all data combined)

```
splisoquant.py -d ont --mode bulk \
--reference GRCh38.chr.fa \
--genedb gencode.v49.annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
--large_output none -p SC.Joglekar -o <output folder>

```

#### Quantification (individual processing)

The extended GTF was obtained using the previous set, but can be also downloaded [here](https://doi.org/10.5281/zenodo.19499423).
Barcode whitelists and their corresponding cell-types are provided [here](https://doi.org/10.5281/zenodo.19499423).

```
splisoquant.py -d ont --mode tenX_v3_split \
--reference GRCh38.chr.fa 
--genedb SC.Joglekar.extended_annotation.gtf --complete_genedb \ 
--fastq <FASTQ files> \
--barcode_whitelist <Barcodes.tsv> \
--barcode2spot <BarcodeClusterAssignments.tsv> \
--read_group barcode_spot --large_output allinfo --no_model_construction \
-p SC.Joglekar -o <output folder>
```
