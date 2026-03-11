---
name: annovar
description: Complete guide for installing ANNOVAR from scratch and running variant annotation on VCF files. Use this skill whenever the user mentions ANNOVAR, variant annotation, annotating VCF files, downloading ANNOVAR databases (refGene, gnomAD, ClinVar, dbSNP, dbNSFP, avsnp), running table_annovar.pl or annotate_variation.pl, converting VCF to ANNOVAR format, or any bioinformatics task involving functional annotation of genetic variants. Trigger this skill for tasks like "install annovar", "annotate my VCF", "download gnomAD for ANNOVAR", "run table_annovar", or "set up ANNOVAR pipeline".
---

# ANNOVAR Skill

ANNOVAR (ANNOtate VARiation) is a Perl-based tool for functional annotation of genetic variants from high-throughput sequencing data. This skill covers installation from scratch through full annotation pipelines.

**Read the reference files when needed:**
- `references/databases.md` — Full list of databases, keywords, and operation types
- `references/troubleshooting.md` — Common errors and fixes

---

## Step 1: Prerequisites

```bash
# Check Perl is installed (required)
perl --version   # Need 5.10+

# Install required Perl modules if missing
cpan LWP::Simple
cpan LWP::Protocol::https
```

---

## Step 2: Download ANNOVAR

ANNOVAR requires **free registration** — it is NOT on GitHub or pip.

1. Go to: https://annovar.openbioinformatics.org/en/latest/user-guide/download/
2. Fill in the registration form (name, institution, email)
3. You will receive an **email** with a download link (usually within minutes)
4. Download the file using **wget** (see below)

> ⚠️ **Do NOT use a browser to download** — Chrome/Edge/Safari will block the download. Always use `wget` or `curl` from the command line.

```bash
# Recommended: use wget with the link from your email
wget "YOUR_DOWNLOAD_LINK" -O annovar.latest.tar.gz

# If the download fails or is interrupted, just run it again — wget will retry automatically
# You can also add --tries and --continue flags for robustness:
wget --tries=10 --continue "YOUR_DOWNLOAD_LINK" -O annovar.latest.tar.gz

# Alternative: use curl
curl -L "YOUR_DOWNLOAD_LINK" -o annovar.latest.tar.gz
```

> 💡 **If the link doesn't work on first try**, simply run the same wget command again. The ANNOVAR server can be slow or occasionally time out — retrying usually succeeds within a few attempts.

```bash
# Extract
tar -xzvf annovar.latest.tar.gz
cd annovar

# Verify contents
ls *.pl
# Should see: annotate_variation.pl  coding_change.pl  convert2annovar.pl
#             retrieve_seq_from_fasta.pl  table_annovar.pl  variants_reduction.pl
```

### Add to PATH (optional but recommended)
```bash
export PATH=$PATH:/path/to/annovar
# Add to ~/.bashrc or ~/.bash_profile to make permanent
echo 'export PATH=$PATH:/path/to/annovar' >> ~/.bashrc
```

---

## Step 3: Directory Structure

After extraction, ANNOVAR has:
```
annovar/
├── annotate_variation.pl   # Core annotation + database download
├── table_annovar.pl        # Multi-protocol wrapper (most commonly used)
├── convert2annovar.pl      # Input format conversion
├── coding_change.pl        # Coding change annotation
├── example/                # Example input files (ex1.avinput, ex2.vcf)
└── humandb/                # Database warehouse (initially near-empty)
```

All downloaded databases go into `humandb/`.

---

## Step 4: Download Databases

Use `annotate_variation.pl -downdb` to download databases into `humandb/`.

### Minimum Required (Gene Annotation)
```bash
cd /path/to/annovar

# hg38 (GRCh38) — recommended for new projects
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar refGene humandb/

# hg19 (GRCh37) — for older datasets
perl annotate_variation.pl -buildver hg19 -downdb -webfrom annovar refGene humandb/
```

### Recommended Clinical/Research Panel
```bash
# For hg38:
BUILDVER=hg38

# Gene annotation
perl annotate_variation.pl -buildver $BUILDVER -downdb -webfrom annovar refGene humandb/

# dbSNP identifiers (latest)
perl annotate_variation.pl -buildver $BUILDVER -downdb -webfrom annovar avsnp151 humandb/

# ClinVar (clinical significance — use latest version)
perl annotate_variation.pl -buildver $BUILDVER -downdb -webfrom annovar clinvar_20240917 humandb/

# gnomAD exome allele frequencies
perl annotate_variation.pl -buildver $BUILDVER -downdb -webfrom annovar gnomad211_exome humandb/

# dbNSFP functional prediction scores (SIFT, PolyPhen2, CADD, etc.)
perl annotate_variation.pl -buildver $BUILDVER -downdb -webfrom annovar dbnsfp47a humandb/
```

> ⚠️ **Large files warning**: gnomAD genome (~200GB) and CADD whole-genome are very large. Only download if needed for WGS analysis.

### Check available databases
```bash
perl annotate_variation.pl -downdb avdblist humandb/ -buildver hg38
# Lists all available databases with size and date
```

---

## Step 5: Input File Formats

### Option A: ANNOVAR native format (.avinput)
Tab-delimited: `chr  start  end  ref  alt  [other columns]`
```
1    948921   948921   T    C
1    1404001  1404001  G    T
X    100000   100001   GG   -
2    234000   234000   -    ATCG
```

### Option B: VCF format (recommended)
Use `-vcfinput` flag with `table_annovar.pl` — no conversion needed.

### Convert VCF to ANNOVAR format (if needed)
```bash
perl convert2annovar.pl -format vcf4 input.vcf > input.avinput
# For multi-sample VCF:
perl convert2annovar.pl -format vcf4 input.vcf -allsample -withfreq > input.avinput
```

---

## Step 6: Run Annotation

### Quick Test (gene annotation only)
```bash
# Using example file
perl table_annovar.pl example/ex2.vcf humandb/ \
    -buildver hg19 \
    -out my_first_anno \
    -protocol refGene \
    -operation g \
    -remove -polish -vcfinput -nastring .
```

### Standard Clinical Panel (hg38, VCF input)
```bash
perl table_annovar.pl input.vcf humandb/ \
    -buildver hg38 \
    -out output_prefix \
    -protocol refGene,avsnp151,clinvar_20240917,gnomad211_exome,dbnsfp47a \
    -operation g,f,f,f,f \
    -remove -polish -vcfinput -nastring . \
    -thread 4
```

### hg19 Multi-database Example
```bash
perl table_annovar.pl input.vcf humandb/ \
    -buildver hg19 \
    -out output_prefix \
    -protocol refGene,avsnp151,clinvar_20240917,gnomad211_exome \
    -operation g,f,f,f \
    -remove -vcfinput -nastring . -polish
```

### Output Files
- `output_prefix.hg38_multianno.txt` — Tab-delimited annotation table
- `output_prefix.hg38_multianno.vcf` — Annotated VCF (with `-vcfinput`)

---

## Operation Types

| Flag | Type | Description |
|------|------|-------------|
| `g`  | gene-based | Variant effect on genes (exonic/intronic/splicing etc.) |
| `r`  | region-based | Overlap with genomic regions (cytoBand, conserved elements) |
| `f`  | filter-based | Lookup in databases (allele freq, ClinVar, dbSNP) |

**Match -protocol and -operation 1:1:**
```
-protocol refGene,avsnp151,clinvar_20240917,gnomad211_exome
-operation g,         f,         f,              f
```

---

## Key Parameters

| Parameter | Description |
|-----------|-------------|
| `-buildver hg38` | Genome build (hg18/hg19/hg38/mm10/etc.) |
| `-protocol` | Comma-separated list of databases |
| `-operation` | Comma-separated g/r/f matching protocol list |
| `-vcfinput` | Accept VCF as input |
| `-remove` | Delete intermediate temp files |
| `-nastring .` | Fill missing values with "." |
| `-polish` | Fix variant representation |
| `-csvout` | Output as CSV instead of TSV |
| `-thread N` | Use N parallel threads |
| `-out PREFIX` | Output file prefix |

---

## Step 7: Interpret Output

Key output columns from `refGene`:
- `Func.refGene` — Location: exonic, intronic, splicing, UTR3, UTR5, intergenic, upstream, downstream
- `Gene.refGene` — Gene symbol
- `ExonicFunc.refGene` — Effect: synonymous/nonsynonymous SNV, frameshift/nonframeshift indel, stopgain, stoploss
- `AAChange.refGene` — cDNA and protein change (e.g., `BRCA1:NM_007294:exon11:c.1234A>G:p.Lys412Arg`)

---

## Common Workflows

### WES/WGS germline variant prioritization
```bash
# 1. Download databases
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar refGene humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar avsnp151 humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar clinvar_20240917 humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar gnomad211_exome humandb/
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar dbnsfp47a humandb/

# 2. Run annotation
perl table_annovar.pl sample.vcf humandb/ \
    -buildver hg38 \
    -out sample_annotated \
    -protocol refGene,avsnp151,clinvar_20240917,gnomad211_exome,dbnsfp47a \
    -operation g,f,f,f,f \
    -remove -polish -vcfinput -nastring . -thread 4
```

### Gene annotation only (fast, no database download needed)
```bash
perl annotate_variation.pl -geneanno -dbtype refGene -buildver hg38 \
    input.avinput humandb/
```

---

## Notes

- ANNOVAR is **free for academic/non-profit use only**; commercial use requires a license
- Registration is required each time you need to re-download the software
- The `humandb/` directory can grow to **hundreds of GB** if you download many databases
- For the latest ClinVar, you may need to build it manually — see `references/databases.md`
- Cite: Wang K, Li M, Hakonarson H. *Nucleic Acids Research* 38:e164, 2010
