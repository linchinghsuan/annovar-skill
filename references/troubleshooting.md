# ANNOVAR Troubleshooting Guide

## Installation Issues

### "perl: command not found"
```bash
# Ubuntu/Debian
sudo apt-get install perl

# CentOS/RHEL
sudo yum install perl

# macOS (usually pre-installed, or via Homebrew)
brew install perl
```

### "Can't locate LWP/Simple.pm"
```bash
# Install via CPAN
cpan install LWP::Simple LWP::Protocol::https

# Or via apt (Ubuntu)
sudo apt-get install libwww-perl

# Or via conda
conda install -c bioconda perl-lwp-simple
```

### "Permission denied" on .pl files
```bash
chmod +x /path/to/annovar/*.pl
```

---

## Database Download Issues

### "Failed" when downloading databases
Usually a network/firewall issue. Try:
```bash
# Test connectivity to ANNOVAR server
curl -I http://www.openbioinformatics.org/annovar/download/

# If HTTPS is blocked by institution, add to ANNOVAR source:
# The -webfrom annovar flag uses http by default (post-2022)
# Some clusters redirect http -> https; if so, use a VPN or request IT help
```

### Database file not found after download
```bash
# Check what's in humandb/
ls humandb/

# Files should be named like: hg38_refGene.txt, hg38_refGeneMrna.fa, etc.
# If partial download, delete and retry:
rm humandb/hg38_refGene*
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar refGene humandb/
```

### Slow download speed
```bash
# Download in background with nohup
nohup perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar gnomad211_exome humandb/ &
# Check progress
tail -f nohup.out
```

---

## Running table_annovar.pl Issues

### "ERROR: the following invalid arguments are specified"
- Check that `-protocol` and `-operation` lists have the **same number of comma-separated items**
- Every protocol needs exactly one corresponding operation (g/r/f)

```bash
# Wrong (3 protocols, 2 operations):
-protocol refGene,avsnp151,clinvar -operation g,f

# Correct:
-protocol refGene,avsnp151,clinvar -operation g,f,f
```

### "Cannot find file humandb/hg38_clinvar_20240917.txt"
The database hasn't been downloaded yet:
```bash
perl annotate_variation.pl -buildver hg38 -downdb -webfrom annovar clinvar_20240917 humandb/
```

### "NOTICE: the --vcfinput flag is set, but..."
Your VCF may have formatting issues:
```bash
# Validate VCF format
grep -v "^#" input.vcf | head -5
# Columns should be: CHROM POS ID REF ALT QUAL FILTER INFO [FORMAT SAMPLE...]
```

### Empty output / no variants annotated
```bash
# Check chromosome naming matches (chr1 vs 1)
# ANNOVAR humandb expects "chr" prefix for hg19/hg38
# If your VCF uses "1" instead of "chr1":
sed 's/^/chr/' input.vcf | grep -v "^chrCHROM" > input_fixed.vcf
# Or use bcftools:
bcftools annotate --rename-chrs chr_name_conv.txt input.vcf > input_fixed.vcf
```

### "Error: the -protocol argument does not specify any known operation"
```bash
# Verify exact spelling of database keyword
# Run avdblist to see all valid keywords:
perl annotate_variation.pl -downdb avdblist humandb/ -buildver hg38
```

---

## Output Issues

### Missing annotation columns (all ".")
- The database file exists but variant positions don't match
- Check genome build consistency: VCF called on hg38 should use `-buildver hg38`
- Check chromosome naming (see above)

### "ExonicFunc" column is empty for some variants
- Only exonic variants get ExonicFunc; intronic/intergenic will be "."
- This is expected behavior

### File too large / memory issues
```bash
# Split VCF by chromosome
for chr in {1..22} X Y; do
    grep -E "^(#|chr${chr}\b)" input.vcf > chr${chr}.vcf
    perl table_annovar.pl chr${chr}.vcf humandb/ \
        -buildver hg38 -out chr${chr}_anno \
        -protocol refGene,avsnp151 -operation g,f \
        -vcfinput -remove -nastring .
done
```

---

## Cluster / HPC Tips

### Run annotation on SLURM
```bash
#!/bin/bash
#SBATCH --job-name=annovar
#SBATCH --cpus-per-task=8
#SBATCH --mem=32G
#SBATCH --time=4:00:00

perl /path/to/annovar/table_annovar.pl input.vcf /path/to/annovar/humandb/ \
    -buildver hg38 \
    -out output_prefix \
    -protocol refGene,avsnp151,clinvar_20240917,gnomad211_exome,dbnsfp47a \
    -operation g,f,f,f,f \
    -thread $SLURM_CPUS_ON_NODE \
    -remove -polish -vcfinput -nastring .
```

### Module load on HPC (if available)
```bash
module load annovar
# ANNOVAR_HOME is set automatically
# Use $ANNOVAR_DATA/{hg19,hg38} for pre-installed databases
```

---

## Docker / Conda Alternative

If you can't register or have installation issues:

### Via Conda (community-maintained)
```bash
conda install -c bioconda annovar
# Note: databases still need to be downloaded via registration
```

### Via Docker
```bash
# Search Docker Hub for community images
docker pull biocontainers/annovar:latest
docker run -v /local/humandb:/humandb biocontainers/annovar \
    table_annovar.pl input.vcf /humandb/ \
    -buildver hg38 -protocol refGene -operation g \
    -vcfinput -out output -remove -nastring .
```

---

## Upgrading ANNOVAR

ANNOVAR itself is updated infrequently. To get the latest version:
1. Re-register at the website (or use your existing registration link if still valid)
2. Download `annovar.latest.tar.gz` again
3. Replace your existing annovar directory (your `humandb/` databases are separate - keep them)

Databases are updated more frequently - re-download specific databases with `-downdb` to get newer versions.
