## Stereo-seq simulation

#### NanoSim model training

Trans-NanoSim model was trained using 2 million reads from StereoSeq mouse data
using [Gencode vM36 mouse annotation](https://www.gencodegenes.org/mouse/release_M36.html).

We used a modified Trans-NanoSim available [here](https://github.com/andrewprzh/lrgasp-simulation).
Model training code was not modified in this version. See details below.
``` 
src/NanoSim/src$ ./read_analysis.py transcriptome \
--ref_g GRCm39.primary_assembly.genome.fa \
--ref_t gencode.vM36.basic.transcritps.fa \
--read StereoSeq_S2_PC1-CID_L_Exome_LW.2M.fastq \
--no_intron_retention \
-o StereoSeq_S2_PC1-CID_L_Exome_LW
```

Model training resulting in the following error rates for simulated data:

- Mismatch rate:    1.44%
- Insertion rate:   1.14%
- Deletion rate:    1.22%
- Total error rate: 3.80%

#### Preparing templates

StereoSeq cDNA templates (full-length molecule sequences without any errors)
were generated using [Gencode vM26 mouse annotation](https://www.gencodegenes.org/mouse/release_M26.html).
Supplementary files are available here.

No cDNA concatenation:
```
python3 simulation/simulate_barcoded.py \
--transcriptome gencode.vM26.basic.transcripts.fasta \
--counts Mouse.ideal_counts.tsv \
--template_count 3000000 --mode stereo \
--barcodes D04620C2.10M.tsv \ 
-o Mouse.stereo10M_3M
```

With cDNA concatenation:
```
python3 simulation/simulate_barcoded.py \
--transcriptome gencode.vM26.basic.transcripts.fasta \
--counts Mouse.ideal_counts.tsv \
--template_count 3000000 --mode stereo --concatenate_templates \
--barcodes D04620C2.10M.tsv \ 
-o Mouse.stereo_concat.10M_3M
```

#### Simulating data
We used template sequences generated at the previous step to simulate ONT-specific
error using Trans-NanoSim.

We used a modified Trans-NanoSim available [here](https://github.com/andrewprzh/lrgasp-simulation).
This version includes a more realistic read truncation method.
It follows empirical read truncation probability at each position on both sides,
rather than  simply following read length distribution (as done originally).
See more details in the [original IsoQuant publication](https://www.nature.com/articles/s41587-022-01565-y#Sec2).

No cDNA concatenation:
```
src/NanoSim/src/simulator.py transcriptome \
--ref_t Mouse.stereo10M_3M.templates.fasta \
--exp Mouse.stereo10M_3M.template_counts.tsv \
--model_prefix  StereoSeq_S2_PC1-CID_L_Exome_LW \
-n 10000000 -b guppy -r cDNA_1D --no_model_ir --aligned_only \
--truncation_mode  none  -t 16 \
--output Mouse.StereoSeq.D04620C2.10M.10M
```


With cDNA concatenation:
```
src/NanoSim/src/simulator.py transcriptome \
--ref_t Mouse.stereo_concat.10M_3M.templates.fasta \
--exp Mouse.stereo_concat.10M_3M.template_counts.tsv \
--model_prefix  StereoSeq_S2_PC1-CID_L_Exome_LW \
-n 10000000 -b guppy -r cDNA_1D --no_model_ir --aligned_only \
--truncation_mode  none  -t 16 \
--output Mouse.StereoSeq_concat.D04620C2.10M.10M
```

#### Running Spl-IsoQuant

Although the simulated reads were generated using only 10 million barcodes,
when running Spl-IsoQuant we used the entire whitelist containing over 450 million barcodes
to obtain relevant statistics.

No cDNA concatenation:
```
splisoquant_detect_barcodes.py  --mode stereoseq_nosplit \
--input Mouse.StereoSeq.D04620C2.10M.10M_aligned_reads.fasta.gz \
--barcodes D04620C2.barcodeToPos.tsv \ 
-o <output prefix>
```

With cDNA concatenation:
```
splisoquant_detect_barcodes.py  --mode stereoseq \
--input Mouse.StereoSeq_concat.D04620C2.10M.10M_aligned_reads.fasta.gz \
--barcodes D04620C2.barcodeToPos.tsv \ 
-o <output prefix>
```

#### Evaluation barcode detection

```
python3 simulation/assess_barcode_quality.py \
--mode <stereo|stereo_split> \
--input <barcoded_reads.tsv> \
--output <output prefix>

```
#### Generating plots

Barcode detection statistics (Fig. 2d,e):
```
python3 simulation/barcodes_sim.py <input.tsv>
```

Precision and recall plot (Fig. 2f, values are copied from barcode assessment output):
```
python3 simulation/prec_recall_sim.py
```

## Visium HD simulation

#### Preparing templates

StereoSeq cDNA templates (full-length molecule sequences without any errors)
were generated using [Gencode vM26 mouse annotation](https://www.gencodegenes.org/mouse/release_M26.html).
Supplementary files are available here. Visium HD barcodes are proprietary and are available on request.

```
python3 simulation/simulate_barcoded.py \
--transcriptome gencode.vM26.basic.transcripts.fasta \
--counts Mouse.ideal_counts.tsv \
--template_count 2000000 --mode  visium_hd \
--barcodes visium_v1_hd.slide1.joint_barcodes.tsv \
-o Mouse.VisiumHD.v1.s1.20M.correct_struct
```

#### Simulating data

We used a modified Trans-NanoSim available [here](https://github.com/andrewprzh/lrgasp-simulation).
This version includes a more realistic read truncation method.
It follows empirical read truncation probability at each position on both sides,
rather than  simply following read length distribution (as done originally).
See more details in the [original IsoQuant publication](https://www.nature.com/articles/s41587-022-01565-y#Sec2).

```
src/NanoSim/src/simulator.py transcriptome \
--ref_t Mouse.VisiumHD.v1.s1.20M.correct_struct.templates.fasta \
--exp Mouse.VisiumHD.v1.s1.20M.correct_struct.template_counts.tsv \
--model_prefix StereoSeq_S2_PC1-CID_L_Exome_LW \
-n 10000000 -b guppy -r cDNA_1D --no_model_ir --aligned_only \
--truncation_mode none --seed 99  --fastq \
--output Mouse.VisiumHD.v1.s1.sep.10M.correct_struct
```

#### Running SpaceRanger

Spaceranger 4.0.1 was used. For conversion, an [official script from 10x](https://github.com/10XGenomics/visium-hd-long-reads/tree/main) was used. 

```
python3 ~/bin/visium-hd-long-reads/long_reads_to_10x_paired.py \
--fastq Mouse.VisiumHD.v1.s1.sep.10M.correct_struct_aligned_reads.fastq \
--threads 1 --sample_name MouseSimC 
```

```
spaceranger-4.0.1/bin/spaceranger count \
--localcores=16 --localmem=128 \
--id MouseSimC --slide=H1-9RJRZBV --area=D1 --create-bam=true \
--transcriptome=refdata-gex-mm10-2020-A \
--cytaimage=CAVG10864_2025-01-21_14-12-28_ter355-c1_H1-9RJRZBV_D1_sample5.tif \
--fastqs=<folder with the coversion output>
```

#### Running Spl-IsoQuant

```
splisoquant_detect_barcodes.py --mode visium_hd \
--input Mouse.VisiumHD.v1.s1.sep.10M.correct_struct_aligned_reads.fasta.gz \
--barcodes visium_v1_hd.slide1.barcodes1.tsv visium_v1_hd.slide1.barcodes2.tsv \ 
-o <output prefix>
```

#### Evaluation barcode detection

Extracting barcode from Spaceranger BAM files: 
```
python3 simulation/bam2tsv.py possorted_genome_bam.bam <output TSV>
```

Evaluating Spaceranger barcodes:
```
python3 simulation/assess_visium_barcodes.py \
--assignment spot <Spaceranger TSV> \
<visium_hd.barcode1_spots.tsv> <visium_hd.barcode2_spots.tsv> <spot_mappings.tsv> 
```

Evaluating Spl-IsoQuant barcodes:
```
python3 simulation/assess_visium_barcodes.py \
--assignment seq <Spl-IsoQuant TSV> \
<visium_hd.barcode1_spots.tsv> <visium_hd.barcode2_spots.tsv> <spot_mappings.tsv> 
```

Visium HD barcode files are proprietary and are available on request.

#### Generating plots

TODO: Lieke could you paste a code for Fig. 5a?