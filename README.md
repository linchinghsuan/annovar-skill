# annovar-skill

A Codex skill for installing ANNOVAR, downloading databases, and annotating VCF files.

## Contents

- `SKILL.md` — main skill definition
- `references/databases.md` — ANNOVAR database guide
- `references/troubleshooting.md` — common errors and fixes

## What this skill covers

This skill helps with:

- installing ANNOVAR from scratch
- downloading annotation databases
- converting VCF to ANNOVAR input format
- running `table_annovar.pl`
- running `annotate_variation.pl`
- annotating human variants with databases such as:
  - refGene
  - avsnp
  - ClinVar
  - gnomAD
  - dbNSFP

## Installation

Clone this repository:

```bash
git clone https://github.com/linchinghsuan/annovar-skill.git
```

Copy it into your Codex skills directory:

```bash
cp -r annovar-skill ~/.codex/skills/annovar
```
