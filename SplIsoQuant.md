## Running Spl-IsoQuant

Spl-IsoQuant can be downloaded from the [official repository](https://github.com/algbio/spl-IsoQuant).
[Version 2.4](https://github.com/algbio/spl-IsoQuant/releases) was used for the analysis.

### Stereo-seq data

Stereo-seq data used in this paper is available on (SRA)[].
Barcode whitelists can be downloaded here.

The correspondence between the data and barcode whitelist is provided in the table below.

TODO: add table

#### Transcript discovery (all data combined)

``` 
splisoquant.py -d ont --mode bulk \
--reference GRCm39.primary_assembly.genome.fa \
--genedb gencode.vM36.basic.annotation.gtf --complete_genedb \
--fastq <FASTQ files>
-p Stereo.ALL -o <output folder> 
```

#### Quantification (individual processing)

The extended GTF was obtained using the previous set, but can be also downloaded here.

``` 
splisoquant.py -d <pacbio|ont> --mode stereoseq \
--reference GRCm39.primary_assembly.genome.fa \
--genedb Stereo.ALL.extended_annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
--barcode_whitelist <Barcodes.tsv> \
--no_model_construction --large_output allinfo \ 
-p Stereo -o <output folder>
```
 

### 10x Visium HD data

TODO: add links

Stereo-seq data used in this paper is available on (SRA)[].
Barcode whitelists and supplementary files are proprietary and are available on request.

#### Barcode detection 

Detecting barcodes sequencing:
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

Alternatively, barcode-to-spot conversion can be done on the fly to skip barcdoe conversion step:

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

 
### 10x Visium data from Lebrigand et al., 2023

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
Barcodes and their corresponding cell-types were extracted for RDS files, which are [available on GEO](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE153859). 

#### Transcript discovery (CBS1 + CBS2)

``` 
splisoquant.py -d ont --mode bulk \
--reference GRCm39.primary_assembly.genome.fa \
--genedb gencode.vM36.basic.annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
 -p Spatial.Lebrigand --large_output none -o <output folder> 
```

#### Quantification (CBS1 and CBS2 separately)

[comment]: <> (TODO: add links)

The extended GTF was obtained using the previous set, but can be also downloaded here.

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

### 10x single-cell data from Joglekar et al., 2024

We used the following ONT data from [Joglekar et al., 2024]():
- Female sample 1:
- 

#### Transcript discovery (all data combined)

```
splisoquant.py -d ont --mode bulk \
--reference GRCh38.chr.fa \
--genedb gencode.v49.annotation.gtf --complete_genedb \
--fastq <FASTQ files> \
--large_output none -p SC.Joglekar -o <output folder>

```

#### Quantification (individual processing)

[comment]: <> (TODO: add links)

The extended GTF was obtained using the previous set, but can be also downloaded here.
Barcode whitelists and their corresponding cell-types are provided here.

```
splisoquant.py -d ont --mode tenX_v3_split \
--reference GRCh38.chr.fa 
--genedb SC.extended_annotation.gtf --complete_genedb \ 
--fastq <FASTQ files> \
--barcode_whitelist <Barcodes.tsv> \
--barcode2spot <BarcodeClusterAssignments.tsv> \
--read_group barcode_spot --large_output allinfo --no_model_construction \
-p SC.Joglekar -o <output folder>
```
