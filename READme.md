# Cannonau genome project 

This repository collects all the commands and script used for the Cannonau genome project. This work is part of the PhD project carried out by Lia Obinu.





## Primary assembly

```
cd Cannonau
git clone https://github.com/LiaOb21/colora.git

cd colora
mkdir resources 
cd resources

mkdir raw_hifi
cd raw_hifi
ln -s ~/Cannonau/30703_Porceddu_Andrea/raw_data/20240708/CAPVS1/hifi_reads/m84140_240705_153855_s2.hifi_reads.all.fastq.gz

cd ..

mkdir raw_hic
cd raw_hic
ln -s ~/Cannonau/30702_Porceddu_Andrea/raw_data/20240625/CAPVS1/240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01_1.fastq.gz
ln -s ~/Cannonau/30702_Porceddu_Andrea/raw_data/20240625/CAPVS1/240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01_2.fastq.gz

cd ..

mkdir oatkDB
cd oatkDB
ln -s ~/software/OatkDB/v20230921/embryophyta_mito.fam
ln -s ~/software/OatkDB/v20230921/embryophyta_mito.fam.h3f
ln -s ~/software/OatkDB/v20230921/embryophyta_mito.fam.h3i
ln -s ~/software/OatkDB/v20230921/embryophyta_mito.fam.h3m
ln -s ~/software/OatkDB/v20230921/embryophyta_mito.fam.h3p
ln -s ~/software/OatkDB/v20230921/embryophyta_pltd.fam
ln -s ~/software/OatkDB/v20230921/embryophyta_pltd.fam.h3f
ln -s ~/software/OatkDB/v20230921/embryophyta_pltd.fam.h3i
ln -s ~/software/OatkDB/v20230921/embryophyta_pltd.fam.h3m
ln -s ~/software/OatkDB/v20230921/embryophyta_pltd.fam.h3p

cd ..

mkdir busco_db
cd busco_db

wget https://busco-data.ezlab.org/v5/data/lineages/eudicots_odb10.2024-01-08.tar.gz
tar -xzf eudicots_odb10.2024-01-08.tar.gz

cd ..

ln -s ~/Barley/decontamination/fcsgx/gxdb

mkdir reference
cd reference

wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/030/704/535/GCF_030704535.1_ASM3070453v1/GCF_030704535.1_ASM3070453v1_genomic.fna.gz

```

Use `primary_assembly/config.yaml` to run [Colora](https://github.com/LiaOb21/colora).

Launch snakemake:
```
cd ~/Cannonau/colora

screen -S Cannonau_primary

mamba activate snakemake

snakemake --software-deployment-method conda
```


N.B. we are using `colora v_1.1.0 - Aghina`.


## Phased assembly

To avoid creating all the environments again, I'm going to run the workflow again inside the same colora directory, but I'll move the results so that I don't lose anything.

```
cd ~/Cannonau/colora 

mamba activate snakemake

snakemake --report

mv results results_primary
mv logs results_primary
mv report.html results_primary
```


The config file I'm going to use for the phased assembly is saved in `phased_assembly/config.yaml`.

```
cd ~/Cannonau/colora
screen -S Cannonau_phased
mamba activate snakemake
snakemake --software-deployment-method conda
```


## Tiara - remove organelle contigs from genome assembly

```
cd Cannonau
mkdir tiara
cd tiara
mkdir primary
mkdir hap1
mkdir hap2

cd primary 

ln -s ~/Cannonau/colora/results_primary/yahs_primary/asm_yahs_scaffolds_final.fa

mamba activate tiara

tiara -i asm_yahs_scaffolds_final.fa -o primary.tiara -t 10 --pr --tf all -m 1000

mv eukarya_asm_yahs_scaffolds_final.fa primary_yahs_tiara.fa

cd ../hap1

ln -s ~/Cannonau/colora/results/yahs_hap1/asm_yahs_scaffolds_final.fa

tiara -i asm_yahs_scaffolds_final.fa -o hap1.tiara -t 10 --pr --tf all -m 1000


mv eukarya_asm_yahs_scaffolds_final.fa hap1_yahs_tiara.fa

cd ../hap2

ln -s ~/Cannonau/colora/results/yahs_hap2/asm_yahs_scaffolds_final.fa

tiara -i asm_yahs_scaffolds_final.fa -o hap2.tiara -t 10 --pr --tf all -m 1000

mv eukarya_asm_yahs_scaffolds_final.fa hap2_yahs_tiara.fa
```

# Treeval

## Primary assembly


### Set up directory structure:

```
cd Cannonau

git clone https://github.com/sanger-tol/treeval.git

cd treeval

mkdir treeval-resources
mkdir -p treeval-resources/gene_alignment_prep/scripts/
cp bin/treeval-dataprep/* treeval-resources/gene_alignment_prep/scripts/
mkdir -p treeval-resources/gene_alignment_prep/raw_fasta/
mkdir -p treeval-resources/gene_alignment_data/dicots/csv_data/
mkdir -p treeval-resources/synteny/dicots/


tree treeval-resources/
---
treeval-resources/
├── gene_alignment_data
│   └── dicots
│       └── csv_data
├── gene_alignment_prep
│   ├── raw_fasta
│   └── scripts
│       ├── GA_csv_gen.py
│       └── GA_data_prep.py
└── synteny
    └── dicots
```

Place the reference genome in the `synteny/dicots` directory (the pipeline doesn't work with gzipped files):

```
cd ~/Cannonau/treeval/treeval-resources/synteny/dicots
cp ~/Cannonau/colora/resources/reference/GCF_030704535.1_ASM3070453v1_genomic.fna.gz ./
gunzip GCF_030704535.1_ASM3070453v1_genomic.fna.gz
```

Now place our assembly in `raw_fasta`. We are going to use the assembly from where we removed the organelles with tiara.

```
cd ~/Cannonau/treeval/treeval-resources/gene_alignment_prep/raw_fasta
ln -s ~/Cannonau/tiara/primary/primary_yahs_tiara.fa VitisVinifera-primary.cdna.fa
```

A this point we have to prep the data.

```
python3 ../scripts/GA_data_prep.py VitisVinifera-primary.cdna.fa ncbi 10
```

This creates a directory that must be moved to its final location:

```
mv VitisVinifera/ ../../gene_alignment_data/dicots/
```

Generate the CSV file:

```
cd ~/Cannonau/treeval/treeval-resources/gene_alignment_prep/

python3 scripts/GA_csv_gen.py ~/Cannonau/treeval/treeval-resources/gene_alignment_data/
```

To check the script worked properly:

```
cd ~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/csv_data

head VitisVinifera.primary-data.csv 
---
org,type,data_file
VitisVinifera.primary,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.primary/cdna/VitisVinifera23cdna.MOD.fa
VitisVinifera.primary,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.primary/cdna/VitisVinifera21cdna.MOD.fa
VitisVinifera.primary,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.primary/cdna/VitisVinifera10cdna.MOD.fa
```

### Hi-C data prep

Illumina HiC read files should be presented in an unmapped CRAM format, each must be accompanied by an index file (.crai) generated by samtools index. If your unmapped HiC reads are in FASTQ format, you should first convert them to CRAM format by using samtools import methods.

IMPORTANT: Here use raw hic NOT pre-filtered reads. All the contacts should be visualised, the mapping used for the scaffolding is good only for visualisation not for manual curation. For manual curation all the links should be included, i.e. NO mapq filter should be applied in the mapping.

```
cd ~/Cannonau/treeval
mkdir hic_data
cd hic_data

cp ~/Cannonau/30702_Porceddu_Andrea/raw_data/20240625/CAPVS1/240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01_*.fastq.gz ./

samtools import -@20 -r ID:240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01 -r CN:arima -r PU:240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01 -r SM:Q31 240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01_1.fastq.gz 240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01_2.fastq.gz -o 240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01.cram

samtools index 240619_A00291_0570_AHCVYWDRX5_1_30702PA0001L01.cram
```

### HiFi data prep

We should convert our fastq.gz reads in fasta

```
cd ~/Cannonau/treeval
mkdir hifi_data
cd hifi_data

ln -s ~/Cannonau/colora/results/reads/hifi/hifi.fastq.gz

mamba activate seqtk

seqtk seq -a hifi.fastq.gz > hifi.fasta

pigz -p100 -c hifi.fasta > hifi.fasta.gz
```

For the `config.yaml` see `treeval/cannonau_primary.yaml`. Put it in `~/Cannonau/treeval`.

### run treeval

```
cd ~/Cannonau/

screen -S treeval_primary

nextflow run treeval/main.nf --input treeval/cannonau_primary.yaml --outdir ~/Cannonau/treeval/primary_out_treeval -c ~/nf_config/ultra2.config -profile singularity,ultra2 -entry RAPID
```

### manual curation

We will do the manual curation using PretextView. A normal laptop is not able to visualise the high resolution map, so we will use the normal map. The file we are looking for is:

`~/Cannonau/treeval/primary_out_treeval/hic_files/primary_1_normal.pretext`

Workflow:

1. sftp the file locally
2. Download PretextView locally:
```
cd software
wget https://github.com/wtsi-hpag/PretextView/releases/download/0.2.5/PretextView_Linux-x86-64.zip

unzip PretextView_Linux-x86-64.zip

#To open pretext view:
cd software
./PretextView
```

3. perform manual curation with pretext view following the [manual](https://gitlab.com/wtsi-grit/rapid-curation/-/blob/main/PretextView-Tutorial.pdf?ref_type=heads).
4. save agp and state before closing pretext.
5. sftp the new generated agp file to the server.


### post curation

```
cd ~/Cannonau/treeval/primary_out_treeval
mkdir post_curation
cd post_curation # place curated AGP file here.

ln -s ~/Cannonau/tiara/primary/primary_yahs_tiara.fa VitisVinifera-primary.cdna.fa
```

We have all the tools installed in the server already. See `treeval.md` for barley (Q31).

```
# activate agp-tpf-utils python env
cd ~/software/agp-tpf-utils
source venv/bin/activate

#run rapid_split on the assembly

cd ~/Cannonau/treeval/primary_out_treeval/post_curation

mamba activate seqtk

perl ~/software/rapid-curation/rapid_split.pl -fa VitisVinifera-primary.cdna.fa

# this script produces VitisVinifera-primary.cdna.fa.tpf

# run pretext-to-tpf

pretext-to-tpf -a VitisVinifera-primary.cdna.fa.tpf -p primary_curated_1_normal.pretext.agp_1 -o VitisVinifera-primary_1_curated.tpf -l INFO -w

# this script produces chrs.csv and VitisVinifera-primary_1_curated.tpf

deactivate # deactivates python env
mamba activate bioperl

perl ~/software/rapid-curation/rapid_join.pl -fa VitisVinifera-primary.cdna.fa -tpf VitisVinifera-primary_1_curated.tpf -csv chrs.csv -out VitisVinifera-primary_curated_final.fa
```


I will rename the directory `treeval-resources` in order to create a new one for the phased assembly and keep both.

```
cd ~/Cannonau/treeval
mv treeval-resources/ primary_treeval-resources
```


## Phased assembly

The manual curation of the phased assembly is done with both haplotypes together at the same time. For this reason, we have to change the fasta headers to add a prefix, and then join them together in a single fasta.

```
cd ~/Cannonau/tiara/hap1
sed 's/^>/>HAP1_/' hap1_yahs_tiara.fa > hap1_yahs_tiara_prefix.fa

cd ~/Cannonau/tiara/hap2
sed 's/^>/>HAP2_/' hap2_yahs_tiara.fa > hap2_yahs_tiara_prefix.fa

cd ~/Cannonau/tiara

cat hap1/hap1_yahs_tiara_prefix.fa hap2/hap2_yahs_tiara_prefix.fa > phased_assembly.fa
```

`phased_assembly.fa` will be our input assembly for treeval.


### Set up directory structure:

```
cd Cannonau/treeval

mkdir treeval-resources
mkdir -p treeval-resources/gene_alignment_prep/scripts/
cp bin/treeval-dataprep/* treeval-resources/gene_alignment_prep/scripts/
mkdir -p treeval-resources/gene_alignment_prep/raw_fasta/
mkdir -p treeval-resources/gene_alignment_data/dicots/csv_data/
mkdir -p treeval-resources/synteny/dicots/


tree treeval-resources/
---
treeval-resources/
├── gene_alignment_data
│   └── dicots
│       └── csv_data
├── gene_alignment_prep
│   ├── raw_fasta
│   └── scripts
│       ├── GA_csv_gen.py
│       └── GA_data_prep.py
└── synteny
    └── dicots
```

Place the reference genome in the `synteny/dicots` directory (the pipeline doesn't work with gzipped files):

```
cd ~/Cannonau/treeval/treeval-resources/synteny/dicots
ln -s ~/Cannonau/treeval/primary_treeval-resources/synteny/dicots/GCF_030704535.1_ASM3070453v1_genomic.fna
```

Now place our assembly in `raw_fasta`. We are going to use the assembly from where we removed the organelles with tiara.

```
cd ~/Cannonau/treeval/treeval-resources/gene_alignment_prep/raw_fasta
ln -s ~/Cannonau/tiara/phased_assembly.fa VitisVinifera-phased.cdna.fa
```

A this point we have to prep the data.

```
python3 ../scripts/GA_data_prep.py VitisVinifera-phased.cdna.fa ncbi 10
```

This creates a directory that must be moved to its final location:

```
mv VitisVinifera/ ../../gene_alignment_data/dicots/
```

Generate the CSV file:

```
cd ~/Cannonau/treeval/treeval-resources/gene_alignment_prep/

python3 scripts/GA_csv_gen.py ~/Cannonau/treeval/treeval-resources/gene_alignment_data/
```

To check the script worked properly:

```
cd ~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/csv_data

head VitisVinifera.phased-data.csv 
---
org,type,data_file
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera131cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera439cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera87cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera43cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera428cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera406cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera296cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera637cdna.MOD.fa
VitisVinifera.phased,cdna,~/Cannonau/treeval/treeval-resources/gene_alignment_data/dicots/VitisVinifera/VitisVinifera.phased/cdna/VitisVinifera461cdna.MOD.fa
```

### Hi-C and HiFi data prep

Since the input data for the phased assembly are exactly the same as the primary assembly, we don't have to repeat these steps, we will just use the previously created input files.

For the `config.yaml` see `treeval/cannonau_phased.yaml`. Put it in `~/Cannonau/treeval`.

### run treeval

```
cd ~/Cannonau/

screen -S treeval_phased

nextflow run treeval/main.nf --input treeval/cannonau_phased.yaml --outdir ~/Cannonau/treeval/phased_out_treeval -c ~/nf_config/ultra2.config -profile singularity,ultra2 -entry RAPID
```

### manual curation

We will do the manual curation using PretextView. A normal laptop is not able to visualise the high resolution map, so we will use the normal map. The file we are looking for is:

`~/Cannonau/treeval/primary_out_treeval/hic_files/phased_1_normal.pretext`

Workflow:

1. sftp the file locally
2. Download PretextView locally:
```
cd software
wget https://github.com/wtsi-hpag/PretextView/releases/download/0.2.5/PretextView_Linux-x86-64.zip

unzip PretextView_Linux-x86-64.zip

#To open pretext view:
cd software
./PretextView
```

3. perform manual curation with pretext view following the [manual](https://gitlab.com/wtsi-grit/rapid-curation/-/blob/main/PretextView-Tutorial.pdf?ref_type=heads).
4. save agp and state before closing pretext.
5. sftp the new generated agp file to the server.



### post curation

```
cd ~/Cannonau/treeval/phased_out_treeval
mkdir post_curation
cd post_curation # place curated AGP file here.

ln -s ~/Cannonau/tiara/phased_assembly.fa VitisVinifera-phased.cdna.fa
```

We have all the tools installed in the server already. See `treeval.md` for barley (Q31).

```
# activate agp-tpf-utils python env
cd ~/software/agp-tpf-utils
source venv/bin/activate

#run rapid_split on the assembly

~/Cannonau/treeval/phased_out_treeval/post_curation

mamba activate seqtk

perl ~/software/rapid-curation/rapid_split.pl -fa VitisVinifera-phased.cdna.fa

# this script produces VitisVinifera-phased.cdna.fa.tpf

# run pretext-to-tpf

pretext-to-tpf -a VitisVinifera-phased.cdna.fa.tpf -p phased_1_normal_after_curation1.pretext.agp_1 -o VitisVinifera-phased_1_curated.tpf -l INFO -w

# this script produces chrs.csv and VitisVinifera-primary_1_curated.tpf

deactivate # deactivates python env
mamba activate bioperl

perl ~/software/rapid-curation/rapid_join.pl -fa VitisVinifera-phased.cdna.fa -tpf VitisVinifera-phased_1_curated_HAP1.tpf -csv chrs_HAP1.csv -out VitisVinifera-phased_curated_final_HAP1.fa

perl ~/software/rapid-curation/rapid_join.pl -fa VitisVinifera-phased.cdna.fa -tpf VitisVinifera-phased_1_curated_HAP2.tpf -csv chrs_HAP2.csv -out VitisVinifera-phased_curated_final_HAP2.fa
```

# Final asssemblyQC

```
cd Cannonau
mkdir final_assemblyQC
cd final_assemblyQC/
mkdir assemblies
cd assemblies

ln -s ~/Cannonau/treeval/primary_out_treeval/post_curation/VitisVinifera-primary_curated_final.fa.curated_primary.no_mt.unscrubbed.fa primary.fasta
ln -s ~/Cannonau/treeval/phased_out_treeval/post_curation/VitisVinifera-phased_curated_final_HAP1.fa.curated_primary.no_mt.unscrubbed.fa hap1.fasta
ln -s ~/Cannonau/treeval/phased_out_treeval/post_curation/VitisVinifera-phased_curated_final_HAP2.fa.curated_primary.no_mt.unscrubbed.fa hap2.fasta

cd ..

wget https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/030/704/535/GCF_030704535.1_ASM3070453v1/GCF_030704535.1_ASM3070453v1_genomic.gff.gz

gunzip GCF_030704535.1_ASM3070453v1_genomic.gff.gz

cp ~/Cannonau/colora/resources/reference/GCF_030704535.1_ASM3070453v1_genomic.fna.gz ./

gunzip GCF_030704535.1_ASM3070453v1_genomic.fna.gz

#liftoff doesn't support compression in gz format

git clone https://github.com/LiaOb21/assemblyQC.git
cd assemblyQC/

screen -S cannonau_assemblyQC_final

./assemblyQC.sh -i ~/Cannonau/final_assemblyQC/assemblies -c ~/software/miniconda3/etc/profile.d/conda.sh -q quast -b busco -m merqury -p ~/software/merqury -l liftoff -d ~/Cannonau/colora/resources/busco_db/eudicots_odb10 -k 21 -r ~/Cannonau/colora/results/reads/hifi/ -f ~/Cannonau/final_assemblyQC/GCF_030704535.1_ASM3070453v1_genomic.fna -g ~/Cannonau/final_assemblyQC/GCF_030704535.1_ASM3070453v1_genomic.gff -s 500 -a NC_007957.1 -e NC_012119.1 -t 50 -o
```

Busco scores are very good: 98.2 complete BUSCOs for the primary assembly and 98.4 for both haplotypes.


# Renaming and reordering the chromosomes

V. vinifera chromosomes are just numbered from 1 to 19. We can see to which chromosome each scaffold corresponds from the output of assemblyQC (liftoff).

This output also tells us the orientation of the chromosomes compared to the reference genome. We are going to reverse the chromosomes to match the reference orientation, as we will need them to be oriented in the same direction to perform the synteny analysis.

| Reference | Primary assembly | Hap 1 | Hap 2 | chr |
|---|---|---|---|---|
| NC_081805 | SUPER_7 | SUPER_5 | SUPER_6 | 1 |
| NC_081806 | SUPER_17 | SUPER_17 | SUPER_17 | 2 |
| NC_081807 | SUPER_15 | SUPER_16 | SUPER_15 | 3 |
| NC_081808 | SUPER_8 | SUPER_9 | SUPER_7 | 4 |
| NC_081809 | SUPER_5 | SUPER_6 | SUPER_11 | 5 |
| NC_081810 | SUPER_13 | SUPER_13 | SUPER_10 | 6 |
| NC_081811 | SUPER_2 | SUPER_2 | SUPER_3 | 7 |
| NC_081812 | SUPER_12 | SUPER_11 | SUPER_12 | 8 |
| NC_081813 | SUPER_6 | SUPER_7 | SUPER_14 | 9 |
| NC_081814 | SUPER_10 | SUPER_8 | SUPER_9 | 10 |
| NC_081815 | SUPER_19 | SUPER_18 | SUPER_18 | 11 |
| NC_081816 | SUPER_9 | SUPER_12 | SUPER_5 | 12 |
| NC_081817 | SUPER_4 | SUPER_3 | SUPER_4 | 13 |
| NC_081818 | SUPER_3 | SUPER_4 | SUPER_2 | 14 |
| NC_081819 | SUPER_16 | SUPER_14 | SUPER_16 | 15 |
| NC_081820 | SUPER_14 | SUPER_15 | SUPER_13 | 16 |
| NC_081821 | SUPER_18 | SUPER_19 | SUPER_19 | 17 |
| NC_081822 | SUPER_1 | SUPER_1 | SUPER_1 | 18 |
| NC_081823 | SUPER_11 | SUPER_10 | SUPER_8 | 19 |


```
cd ~/Cannonau/final_assemblyQC/assemblies

sed -e 's/^>SUPER_7$/>chr_01/' \
    -e 's/^>SUPER_17$/>chr_02/' \
    -e 's/^>SUPER_15$/>chr_03/' \
    -e 's/^>SUPER_8$/>chr_04/' \
    -e 's/^>SUPER_5$/>chr_05/' \
    -e 's/^>SUPER_13$/>chr_06/' \
    -e 's/^>SUPER_2$/>chr_07/' \
    -e 's/^>SUPER_12$/>chr_08/' \
    -e 's/^>SUPER_6$/>chr_09/' \
    -e 's/^>SUPER_10$/>chr_10/' \
    -e 's/^>SUPER_19$/>chr_11/' \
    -e 's/^>SUPER_9$/>chr_12/' \
    -e 's/^>SUPER_4$/>chr_13/' \
    -e 's/^>SUPER_3$/>chr_14/' \
    -e 's/^>SUPER_16$/>chr_15/' \
    -e 's/^>SUPER_14$/>chr_16/' \
    -e 's/^>SUPER_18$/>chr_17/' \
    -e 's/^>SUPER_1$/>chr_18/' \
    -e 's/^>SUPER_11$/>chr_19/' \
    primary.fasta > VitisVinifera_cannonau_CAPVS1_primary.fa
```

Reorder:
```
nano chr_order.txt
---
chr_01
chr_02
chr_03
chr_04
chr_05
chr_06
chr_07
chr_08
chr_09
chr_10
chr_11
chr_12
chr_13
chr_14
chr_15
chr_16
chr_17
chr_18
chr_19

samtools faidx -r chr_order.txt VitisVinifera_cannonau_CAPVS1_primary.fa > VitisVinifera_cannonau_CAPVS1_primary_FINAL.fa
```



```
cd ~/Cannonau/final_assemblyQC/assemblies
sed -e 's/^>SUPER_5$/>chr_01/' \
    -e 's/^>SUPER_17$/>chr_02/' \
    -e 's/^>SUPER_16$/>chr_03/' \
    -e 's/^>SUPER_9$/>chr_04/' \
    -e 's/^>SUPER_6$/>chr_05/' \
    -e 's/^>SUPER_13$/>chr_06/' \
    -e 's/^>SUPER_2$/>chr_07/' \
    -e 's/^>SUPER_11$/>chr_08/' \
    -e 's/^>SUPER_7$/>chr_09/' \
    -e 's/^>SUPER_8$/>chr_10/' \
    -e 's/^>SUPER_18$/>chr_11/' \
    -e 's/^>SUPER_12$/>chr_12/' \
    -e 's/^>SUPER_3$/>chr_13/' \
    -e 's/^>SUPER_4$/>chr_14/' \
    -e 's/^>SUPER_14$/>chr_15/' \
    -e 's/^>SUPER_15$/>chr_16/' \
    -e 's/^>SUPER_19$/>chr_17/' \
    -e 's/^>SUPER_1$/>chr_18/' \
    -e 's/^>SUPER_10$/>chr_19/' \
    hap1.fasta > VitisVinifera_cannonau_CAPVS1_haplotype1.fa
```

Reorder:

```
samtools faidx -r chr_order.txt VitisVinifera_cannonau_CAPVS1_haplotype1.fa > ordered_chrs_hap1.fa

mamba activate seqkit
seqkit grep -r -p "HAP1" VitisVinifera_cannonau_CAPVS1_haplotype1.fa > unplaced_hap1.fa

cat ordered_chrs_hap1.fa unplaced_hap1.fa > VitisVinifera_cannonau_CAPVS1_haplotype1_FINAL.fa
```

```
sed -e 's/^>SUPER_6$/>chr_01/' \
    -e 's/^>SUPER_17$/>chr_02/' \
    -e 's/^>SUPER_15$/>chr_03/' \
    -e 's/^>SUPER_7$/>chr_04/' \
    -e 's/^>SUPER_11$/>chr_05/' \
    -e 's/^>SUPER_10$/>chr_06/' \
    -e 's/^>SUPER_3$/>chr_07/' \
    -e 's/^>SUPER_12$/>chr_08/' \
    -e 's/^>SUPER_14$/>chr_09/' \
    -e 's/^>SUPER_9$/>chr_10/' \
    -e 's/^>SUPER_18$/>chr_11/' \
    -e 's/^>SUPER_5$/>chr_12/' \
    -e 's/^>SUPER_4$/>chr_13/' \
    -e 's/^>SUPER_2$/>chr_14/' \
    -e 's/^>SUPER_16$/>chr_15/' \
    -e 's/^>SUPER_13$/>chr_16/' \
    -e 's/^>SUPER_19$/>chr_17/' \
    -e 's/^>SUPER_1$/>chr_18/' \
    -e 's/^>SUPER_8$/>chr_19/' \
    hap2.fasta > VitisVinifera_cannonau_CAPVS1_haplotype2.fa
```

Reorder:

```
samtools faidx -r chr_order.txt VitisVinifera_cannonau_CAPVS1_haplotype2.fa > ordered_chrs_hap2.fa

mamba activate seqkit
seqkit grep -r -p "HAP2" VitisVinifera_cannonau_CAPVS1_haplotype2.fa > unplaced_hap2.fa

cat ordered_chrs_hap2.fa unplaced_hap2.fa > VitisVinifera_cannonau_CAPVS1_haplotype2_FINAL.fa
```

N.B. we need to modify `*_curated_final.fa.chromosome.list.csv` for the assembly submission to match names and order of the chromosomes.



# blob (snail plots)

## primary
```
cd ~/Cannonau/final_assemblyQC/

mkdir blob
cd blob

mamba activate btk

nano primary.yaml
---
assembly:
  alias: Cannonau_CAPVS1_primary
  record_type: chromosome
taxon:
  name: Vitis vinifera
  taxid: 29760
```

```
blobtools create \
    --fasta ~/Cannonau/final_assemblyQC/assemblies/primary.fasta \
    --meta primary.yaml \
    ./Cannonau_CAPVS1_primary
```

```
ln -s ~/Cannonau/final_assemblyQC/assemblyQC/assembly_QC_out/busco_ok/primary_busco/run_eudicots_odb10/full_table.tsv primary_full_table.tsv

blobtools add \
    --busco primary_full_table.tsv \
    ./Cannonau_CAPVS1_primary
```


sftp blobdir locally

```
mamba activate btk

blobtk plot -v snail -d Cannonau_CAPVS1_primary/ -o Cannonau_CAPVS1_primary_snail.png
```

## hap1

```
cd ~/Cannonau/final_assemblyQC/blob

nano hap1.yaml
---
assembly:
  alias: Cannonau_CAPVS1_hap1
  record_type: chromosome
taxon:
  name: Vitis vinifera
  taxid: 29760
```

```
blobtools create \
    --fasta ~/Cannonau/final_assemblyQC/assemblies/hap1.fasta \
    --meta hap1.yaml \
    ./Cannonau_CAPVS1_hap1
```

```
ln -s ~/Cannonau/final_assemblyQC/assemblyQC/assembly_QC_out/busco_ok/hap1_busco/run_eudicots_odb10/full_table.tsv hap1_full_table.tsv

blobtools add \
    --busco hap1_full_table.tsv \
    ./Cannonau_CAPVS1_hap1
```


sftp blobdir locally

```
mamba activate btk

blobtk plot -v snail -d Cannonau_CAPVS1_hap1/ -o Cannonau_CAPVS1_hap1_snail.png
```

## hap2

```
cd ~/Cannonau/final_assemblyQC/blob

nano hap2.yaml
---
assembly:
  alias: Cannonau_CAPVS1_hap2
  record_type: chromosome
taxon:
  name: Vitis vinifera
  taxid: 29760
```

```
blobtools create \
    --fasta ~/Cannonau/final_assemblyQC/assemblies/hap2.fasta \
    --meta hap2.yaml \
    ./Cannonau_CAPVS1_hap2
```

```
ln -s ~/Cannonau/final_assemblyQC/assemblyQC/assembly_QC_out/busco_ok/hap2_busco/run_eudicots_odb10/full_table.tsv hap2_full_table.tsv

blobtools add \
    --busco hap2_full_table.tsv \
    ./Cannonau_CAPVS1_hap2
```


sftp blobdir locally

```
mamba activate btk

blobtk plot -v snail -d Cannonau_CAPVS1_hap2/ -o Cannonau_CAPVS1_hap2_snail.png
```



# Synteny rearrangements

Before performing the synteny, we need to reverse complement those chromosomes that are not in the same orientation as the reference, otherwise SyRi won't work.

For the primary assembly, we need to reverse complement the following chromosomes: 02, 03, 06, 08, 09, 10, 11, 13, 18. We can check this from `liftoff_combine.py` results from assemblyQC. We can use samtools for extracting these chromosomes and seqkit to reverse complement them.

```
cd Cannonau 
mkdir synteny
cd synteny

mkdir reorder
cd reorder
mkdir primary

ln -s ~/Cannonau/final_assemblyQC/assemblies/VitisVinifera_cannonau_CAPVS1_primary_FINAL.fa

nano chrs_to_reverse.txt
---
chr_02
chr_03
chr_06
chr_08
chr_09
chr_10
chr_11
chr_13
chr_18

samtools faidx -r chrs_to_reverse.txt VitisVinifera_cannonau_CAPVS1_primary_FINAL.fa > chrs_to_rev_primary.fa

mamba activate seqkit

seqkit seq chrs_to_rev_primary.fa -r -p -t DNA > reversed_primary.fa

# extract the all the other chromosomes from the original assembly

nano chrs_forward.txt
---
chr_01
chr_04
chr_05
chr_07
chr_12
chr_14
chr_15
chr_16
chr_17
chr_19

mamba deactivate 

samtools faidx -r chrs_forward.txt VitisVinifera_cannonau_CAPVS1_primary_FINAL.fa > forward_primary.fa

cat forward_primary.fa reversed_primary.fa > all_forward_primary.fa

nano chr_order.txt
---
chr_01
chr_02
chr_03
chr_04
chr_05
chr_06
chr_07
chr_08
chr_09
chr_10
chr_11
chr_12
chr_13
chr_14
chr_15
chr_16
chr_17
chr_18
chr_19

samtools faidx -r chr_order.txt all_forward_primary.fa > primary_syn.fa
```

For the two haplotypes, we are going to use only the chromosome-length scaffolds

```
cd ~/Cannonau/synteny/reorder

mkdir hap1 
cd hap1

ln -s ~/Cannonau/final_assemblyQC/assemblies/ordered_chrs_hap1.fa

nano chrs_to_reverse.txt
---
chr_01
chr_02
chr_04
chr_08
chr_09
chr_10
chr_13
chr_15
chr_18

samtools faidx -r chrs_to_reverse.txt ordered_chrs_hap1.fa > chrs_to_rev_hap1.fa

mamba activate seqkit

seqkit seq chrs_to_rev_hap1.fa -r -p -t DNA > reversed_hap1.fa

# extract the all the other chromosomes from the original assembly

nano chrs_forward.txt
---
chr_03
chr_05
chr_06
chr_07
chr_11
chr_12
chr_14
chr_16
chr_17
chr_19

mamba deactivate 

samtools faidx -r chrs_forward.txt ordered_chrs_hap1.fa > forward_hap1.fa

cat forward_hap1.fa reversed_hap1.fa > all_forward_hap1.fa

nano chr_order.txt
---
chr_01
chr_02
chr_03
chr_04
chr_05
chr_06
chr_07
chr_08
chr_09
chr_10
chr_11
chr_12
chr_13
chr_14
chr_15
chr_16
chr_17
chr_18
chr_19

samtools faidx -r chr_order.txt all_forward_hap1.fa > hap1_syn.fa
```



```
cd ~/Cannonau/synteny/reorder

mkdir hap2 
cd hap2

ln -s ~/Cannonau/final_assemblyQC/assemblies/ordered_chrs_hap2.fa

nano chrs_to_reverse.txt
---
chr_01
chr_02
chr_04
chr_08
chr_09
chr_10
chr_13
chr_15
chr_18

samtools faidx -r chrs_to_reverse.txt ordered_chrs_hap2.fa > chrs_to_rev_hap2.fa

mamba activate seqkit

seqkit seq chrs_to_rev_hap2.fa -r -p -t DNA > reversed_hap2.fa

# extract the all the other chromosomes from the original assembly

nano chrs_forward.txt
---
chr_03
chr_05
chr_06
chr_07
chr_11
chr_12
chr_14
chr_16
chr_17
chr_19

mamba deactivate 

samtools faidx -r chrs_forward.txt ordered_chrs_hap2.fa > forward_hap2.fa

cat forward_hap2.fa reversed_hap2.fa > all_forward_hap2.fa

nano chr_order.txt
---
chr_01
chr_02
chr_03
chr_04
chr_05
chr_06
chr_07
chr_08
chr_09
chr_10
chr_11
chr_12
chr_13
chr_14
chr_15
chr_16
chr_17
chr_18
chr_19

samtools faidx -r chr_order.txt all_forward_hap2.fa > hap2_syn.fa
```


Now, we have to rename the chromosomes in the reference genome, as SyRi needs them to be named and oriented in the same way

```
cd ~/Cannonau/synteny/reorder

ln -s ~/Cannonau/final_assemblyQC/GCF_030704535.1_ASM3070453v1_genomic.fna

sed -e 's/^>NC_081805.1/>chr_01/' \
    -e 's/^>NC_081806.1/>chr_02/' \
    -e 's/^>NC_081807.1/>chr_03/' \
    -e 's/^>NC_081808.1/>chr_04/' \
    -e 's/^>NC_081809.1/>chr_05/' \
    -e 's/^>NC_081810.1/>chr_06/' \
    -e 's/^>NC_081811.1/>chr_07/' \
    -e 's/^>NC_081812.1/>chr_08/' \
    -e 's/^>NC_081813.1/>chr_09/' \
    -e 's/^>NC_081814.1/>chr_10/' \
    -e 's/^>NC_081815.1/>chr_11/' \
    -e 's/^>NC_081816.1/>chr_12/' \
    -e 's/^>NC_081817.1/>chr_13/' \
    -e 's/^>NC_081818.1/>chr_14/' \
    -e 's/^>NC_081819.1/>chr_15/' \
    -e 's/^>NC_081820.1/>chr_16/' \
    -e 's/^>NC_081821.1/>chr_17/' \
    -e 's/^>NC_081822.1/>chr_18/' \
    -e 's/^>NC_081823.1/>chr_19/' \
    GCF_030704535.1_ASM3070453v1_genomic.fna > refernce.fa

# remove stuff after first space in fasta heades

sed 's/\(>[^ ]*\).*/\1/' refernce.fa > modified_ref.fasta

## keep only chromosomes (remove organelles)

nano chr_order.txt
---
chr_01
chr_02
chr_03
chr_04
chr_05
chr_06
chr_07
chr_08
chr_09
chr_10
chr_11
chr_12
chr_13
chr_14
chr_15
chr_16
chr_17
chr_18
chr_19

samtools faidx -r chr_order.txt modified_ref.fasta > reference_syn.fa
```

Okay, now all our `*_syn.fa` files are ready for the synteny analysis


I want to visualise synteny between the reference, the primary assembly and the two haplotypes. 


Install syri
```
mamba create -n syri_env -c bioconda syri
mamba activate syri_env
mamba install plotsr # for visualisation
```

We need to run multiple alignment with Minimap2:
- reference vs primary
- hap1 vs hap2


```
cd ~/Cannonau/synteny

screen -S ref_primary
mamba activate minimap2
minimap2 -ax asm5 --eqx reorder/reference_syn.fa reorder/primary/primary_syn.fa -t 50 > ref_primary.sam


screen -S hap1_hap2
mamba activate minimap2
minimap2 -ax asm5 --eqx reorder/hap1/hap1_syn.fa reorder/hap2/hap2_syn.fa -t 50 > hap1_hap2.sam


samtools view -b ref_primary.sam > ref_primary.bam
samtools sort ref_primary.bam > ref_primary_sort.bam

samtools view -b primary_hap1.sam > primary_hap1.bam
samtools sort primary_hap1.bam > primary_hap1_sort.bam

samtools view -b hap1_hap2.sam > hap1_hap2.bam
samtools sort hap1_hap2.bam > hap1_hap2_sort.bam

mamba activate syri_env

syri -c ref_primary_sort.bam -r reorder/reference_syn.fa -q reorder/primary/primary_syn.fa -F B --prefix ref_primary --nc 19

syri -c primary_hap1_sort.bam -r reorder/primary/primary_syn.fa -q reorder/hap1/hap1_syn.fa -F B --prefix primary_hap1 --nc 19

syri -c hap1_hap2_sort.bam -r reorder/hap1/hap1_syn.fa -q reorder/hap2/hap2_syn.fa -F B --prefix hap1_hap2 --nc 19
```

We are now ready for the plot:

```
cd ~/Cannonau/synteny
mamba activate syri_env

nano genomes.txt
---
nano ref_vs_primary.txt
---
#file name  tags
reorder/reference_syn.fa  reference lw:1.5
reorder/primary/primary_syn.fa  primary lw:1.5

plotsr \
    --sr ref_primarysyri.out \
    --genomes ref_vs_primary.txt \
    -o ref_vs_primary_plot.svg -b svg

nano hap1_vs_hap2.txt
---
#file name  tags
reorder/hap1/hap1_syn.fa  hap1  lw:1.5
reorder/hap2/hap2_syn.fa  hap2  lw:1.5

plotsr \
    --sr hap1_hap2syri.out \
    --genomes hap1_vs_hap2.txt \
    -o hap1_vs_hap2.svg -b svg
```


# Annotation stats from Liftoff

Modify the output of liftoff to match the chromosome names:

```
awk 'BEGIN {FS=OFS="\t"} {
    sub(/^SUPER_7$/, "chr_01", $1);
    sub(/^SUPER_17$/, "chr_02", $1);
    sub(/^SUPER_15$/, "chr_03", $1);
    sub(/^SUPER_8$/, "chr_04", $1);
    sub(/^SUPER_5$/, "chr_05", $1);
    sub(/^SUPER_13$/, "chr_06", $1);
    sub(/^SUPER_2$/, "chr_07", $1);
    sub(/^SUPER_12$/, "chr_08", $1);
    sub(/^SUPER_6$/, "chr_09", $1);
    sub(/^SUPER_10$/, "chr_10", $1);
    sub(/^SUPER_19$/, "chr_11", $1);
    sub(/^SUPER_9$/, "chr_12", $1);
    sub(/^SUPER_4$/, "chr_13", $1);
    sub(/^SUPER_3$/, "chr_14", $1);
    sub(/^SUPER_16$/, "chr_15", $1);
    sub(/^SUPER_14$/, "chr_16", $1);
    sub(/^SUPER_18$/, "chr_17", $1);
    sub(/^SUPER_1$/, "chr_18", $1);
    sub(/^SUPER_11$/, "chr_19", $1);
    print
}' primary.fasta_liftoff_out.gff3 > adj_primary.fasta_liftoff_out.gff3


awk 'BEGIN {FS=OFS="\t"} {
    sub(/^NC_081805.1/, "chr_01", $1);
    sub(/^NC_081806.1/, "chr_02", $1);
    sub(/^NC_081807.1/, "chr_03", $1);
    sub(/^NC_081808.1/, "chr_04", $1);
    sub(/^NC_081809.1/, "chr_05", $1);
    sub(/^NC_081810.1/, "chr_06", $1);
    sub(/^NC_081811.1/, "chr_07", $1);
    sub(/^NC_081812.1/, "chr_08", $1);
    sub(/^NC_081813.1/, "chr_09", $1);
    sub(/^NC_081814.1/, "chr_10", $1);
    sub(/^NC_081815.1/, "chr_11", $1);
    sub(/^NC_081816.1/, "chr_12", $1);
    sub(/^NC_081817.1/, "chr_13", $1);
    sub(/^NC_081818.1/, "chr_14", $1);
    sub(/^NC_081819.1/, "chr_15", $1);
    sub(/^NC_081820.1/, "chr_16", $1);
    sub(/^NC_081821.1/, "chr_17", $1);
    sub(/^NC_081822.1/, "chr_18", $1);
    sub(/^NC_081823.1/, "chr_19", $1);
    print
}' GCF_030704535.1_ASM3070453v1_genomic.gff > adj_reference.gff


awk 'BEGIN {FS=OFS="\t"} {
    sub(/^SUPER_5$/, "chr_01", $1);
    sub(/^SUPER_17$/, "chr_02", $1);
    sub(/^SUPER_16$/, "chr_03", $1);
    sub(/^SUPER_9$/, "chr_04", $1);
    sub(/^SUPER_6$/, "chr_05", $1);
    sub(/^SUPER_13$/, "chr_06", $1);
    sub(/^SUPER_2$/, "chr_07", $1);
    sub(/^SUPER_11$/, "chr_08", $1);
    sub(/^SUPER_7$/, "chr_09", $1);
    sub(/^SUPER_8$/, "chr_10", $1);
    sub(/^SUPER_18$/, "chr_11", $1);
    sub(/^SUPER_12$/, "chr_12", $1);
    sub(/^SUPER_3$/, "chr_13", $1);
    sub(/^SUPER_4$/, "chr_14", $1);
    sub(/^SUPER_14$/, "chr_15", $1);
    sub(/^SUPER_15$/, "chr_16", $1);
    sub(/^SUPER_19$/, "chr_17", $1);
    sub(/^SUPER_1$/, "chr_18", $1);
    sub(/^SUPER_10$/, "chr_19", $1);
    print
}' hap1.fasta_liftoff_out.gff3 > adj_hap1.fasta_liftoff_out.gff3



awk 'BEGIN {FS=OFS="\t"} {
    sub(/^SUPER_6$/, "chr_01", $1);
    sub(/^SUPER_17$/, "chr_02", $1);
    sub(/^SUPER_15$/, "chr_03", $1);
    sub(/^SUPER_7$/, "chr_04", $1);
    sub(/^SUPER_11$/, "chr_05", $1);
    sub(/^SUPER_10$/, "chr_06", $1);
    sub(/^SUPER_3$/, "chr_07", $1);
    sub(/^SUPER_12$/, "chr_08", $1);
    sub(/^SUPER_14$/, "chr_09", $1);
    sub(/^SUPER_9$/, "chr_10", $1);
    sub(/^SUPER_18$/, "chr_11", $1);
    sub(/^SUPER_5$/, "chr_12", $1);
    sub(/^SUPER_4$/, "chr_13", $1);
    sub(/^SUPER_2$/, "chr_14", $1);
    sub(/^SUPER_16$/, "chr_15", $1);
    sub(/^SUPER_13$/, "chr_16", $1);
    sub(/^SUPER_19$/, "chr_17", $1);
    sub(/^SUPER_1$/, "chr_18", $1);
    sub(/^SUPER_8$/, "chr_19", $1);
    print
}' hap2.fasta_liftoff_out.gff3 > adj_hap2.fasta_liftoff_out.gff3

```

Then, follow `extract_stats_liftoff.ipynb`.
