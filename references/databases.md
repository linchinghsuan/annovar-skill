# ANNOVAR Database Reference

## Gene-based Databases (operation: `g`)

| Keyword | Description | Build |
|---------|-------------|-------|
| `refGene` | NCBI RefSeq gene annotation (recommended) | hg19/hg38 |
| `refGeneWithVer` | RefSeq with version numbers in transcript IDs | hg19/hg38 |
| `ensGene` | Ensembl gene annotation (Gencode v43) | hg19/hg38 |
| `knownGene` | UCSC KnownGene | hg19/hg38 |

**Tip:** Use `refGeneWithVer` when you need exact transcript versioning (e.g., `NM_007294.4`).

---

## Filter-based Databases (operation: `f`)

### Population Allele Frequency

| Keyword | Description | Variants | Build |
|---------|-------------|----------|-------|
| `gnomad211_exome` | gnomAD v2.1.1 exome (125k individuals) | 17.2M | hg19/hg38 |
| `gnomad211_genome` | gnomAD v2.1.1 genome (16k individuals) | 262M | hg19/hg38 |
| `gnomad312_genome` | gnomAD v3.1.2 genome (76k individuals) | - | hg38 |
| `gnomad40_exome` | gnomAD v4.0 exome (730k individuals) | - | hg38 |
| `gnomad40_genome` | gnomAD v4.0 genome (76k individuals) | - | hg38 |
| `avsnp151` | dbSNP 151 (left-normalized, allele-split) | - | hg19/hg38 |
| `avsnp150` | dbSNP 150 | - | hg19/hg38 |
| `1000g2015aug_all` | 1000 Genomes phase 3 (all populations) | - | hg19/hg38 |
| `1000g2015aug_eas` | 1000G East Asian | - | hg19/hg38 |
| `1000g2015aug_eur` | 1000G European | - | hg19/hg38 |
| `exac03` | ExAC v0.3 (60k exomes) | - | hg19/hg38 |
| `esp6500siv2_all` | NHLBI ESP 6500 exomes | - | hg19/hg38 |

### Clinical Significance

| Keyword | Description | Build |
|---------|-------------|-------|
| `clinvar_20240917` | ClinVar (Sep 2024) - germline + somatic | hg19/hg38 |
| `clinvar_20221231` | ClinVar (Dec 2022) | hg19/hg38 |
| `intervar_20180118` | InterVar ACMG/AMP 2015 criteria (missense) | hg19/hg38 |

**ClinVar output columns:** `CLNSIG` (significance), `CLNDN` (disease name), `CLNREVSTAT` (review status)

### Functional Prediction Scores (dbNSFP)

| Keyword | Description | Build |
|---------|-------------|-------|
| `dbnsfp47a` | dbNSFP v4.7a - includes AlphaMissense, ESM1b, SIFT, PolyPhen2, CADD, REVEL, MutationTaster, FATHMM, MetaSVM, MetaLR, VEST4, MutPred, MVP | hg19/hg38 |
| `dbnsfp47c` | dbNSFP v4.7c (conservative subset) | hg19/hg38 |
| `dbnsfp42a` | dbNSFP v4.2a | hg19/hg38 |
| `ljb26_all` | Older LJB scores (SIFT, PolyPhen2, LRT, MT, GERP++, etc.) | hg19/hg38 |

**Key scores in dbnsfp47a:**
- `SIFT_score` - <0.05 = deleterious
- `Polyphen2_HDIV_score` - >0.909 = probably damaging
- `CADD_phred` - >20 = top 1% deleterious, >30 = top 0.1%
- `REVEL_score` - >0.5 likely pathogenic, >0.75 strong evidence
- `AlphaMissense_score` - >0.564 = likely pathogenic
- `MutationTaster_pred` - A=polymorphism, D=disease-causing

### Splicing

| Keyword | Description | Build |
|---------|-------------|-------|
| `regsnpintron` | regSNP-intron splicing site prediction | hg19/hg38 |

---

## Region-based Databases (operation: `r`)

| Keyword | Description | Build |
|---------|-------------|-------|
| `cytoBand` | Cytogenetic band locations | hg19/hg38 |
| `genomicSuperDups` | Segmental duplications | hg19/hg38 |
| `phastConsElements46way` | Conserved elements (46-way vertebrate) | hg19 |
| `gwasCatalog` | GWAS catalog associations | hg19/hg38 |
| `dgvMerged` | Database of Genomic Variants (structural) | hg19/hg38 |

---

## Download Commands Quick Reference

```bash
# Always run from inside the annovar/ directory
cd /path/to/annovar

# Gene annotation
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar refGene humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar refGeneWithVer humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar ensGene humandb/

# Allele frequency
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar avsnp151 humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar gnomad211_exome humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar gnomad40_exome humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar 1000g2015aug humandb/

# Clinical
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar clinvar_20240917 humandb/

# Functional predictions
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar dbnsfp47a humandb/

# Region-based
perl annotate_variation.pl -buildver hg38 -downdb cytoBand humandb/   # no -webfrom for UCSC databases
```

---

## Building Latest ClinVar Manually

When the ANNOVAR server's ClinVar lags behind NCBI:

```bash
# Download from NCBI (hg38)
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh38/clinvar_20240917.vcf.gz
wget ftp://ftp.ncbi.nlm.nih.gov/pub/clinvar/vcf_GRCh38/clinvar_20240917.vcf.gz.tbi
gunzip clinvar_20240917.vcf.gz

# Prepare for ANNOVAR
perl prepare_annovar_user.pl -dbtype clinvar2 clinvar_20240917.vcf \
    -out hg38_clinvar_20240917_raw.txt

# Index
perl index_annovar.pl hg38_clinvar_20240917_raw.txt \
    -out hg38_clinvar_20240917.txt \
    -comment comment_clinvar_20240917.txt

# Download the comment file (needed for column headers)
# From: https://annovar.openbioinformatics.org/en/latest/user-guide/filter/
```

---

## Recommended Database Combinations

### Rare Disease / Mendelian Genetics
```
-protocol refGeneWithVer,avsnp151,clinvar_20240917,gnomad211_exome,dbnsfp47a,intervar_20180118
-operation g,f,f,f,f,f
```

### Cancer Somatic Variants
```
-protocol refGene,avsnp151,clinvar_20240917,cosmic70,gnomad211_exome
-operation g,f,f,f,f
```

### GWAS/Population Studies
```
-protocol refGene,avsnp151,1000g2015aug_all,gnomad211_exome,cytoBand
-operation g,f,f,f,r
```

### Quick Annotation (gene + ClinVar only)
```
-protocol refGene,clinvar_20240917
-operation g,f
```
