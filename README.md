# annovar-skill

A skill for installing ANNOVAR, downloading databases, and annotating VCF files.

## Demo

A short demonstration of annotating a VCF file using `annovar.skill`.

▶️ Watch the demo video:  
https://raw.githubusercontent.com/linchinghsuan/annovar-skill/main/annovar.mov


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

## Example Setup

Clone this repository:

```bash
git clone https://github.com/linchinghsuan/annovar-skill.git
```

For example, if you are using a Codex local skill environment, you can place the skill in the skills directory:

```bash
cp -r annovar-skill ~/.codex/skills/annovar
```
