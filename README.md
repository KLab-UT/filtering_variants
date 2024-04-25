# Genomics Pipeline Intro

This repository provides an introduction to basic techniques in variant calling and filtration. Specifically, participants will get a refresher on creating a VCF file from a BAM file and will learn about the approaches and consequences of filtering variants within a VCF. The purpose of this introduction is to help individuals new to genomics understand the basics of a critical step in bioinformatics; it is not to present every possible option in variant filtration.

---

# Contents

-   [Objectives](#objectives)
-   [Variant Call Format](#variant-call-format)
-   [Variant Filtration](#variant-filtration)
-   [Exercise](#exercise)

---

# <a name="objectives"></a>
# Objectives

-  Understand the basics of calling variants from an alignment file
-  Understand why and how to filter variants
---

# <a name="getting-set-up"></a>
# Getting set up
If you are here as a UTU student taking BIOL 4310, you should do the following:

1.  Login to your [Github](https://github.com/) account.

1.  Fork [this repository](https://github.com/KLab-UT/filtering_variants), by
    clicking the 'Fork' button on the upper right of the page.

    After a few seconds, you should be looking at *your*
    copy of the repo in your own Github account.

1.  Click the 'Clone or download' button, and copy the URL of the repo via the
    'copy to clipboard' button. **note: if you have an SSH key with your github account, make sure you select the ```SSH``` tab**

1.  In your terminal, navigate to where you want to keep this repo (you can
    always move it later, so just your home directory is fine). Then type:

        $ git clone the-url-you-just-copied

    and hit enter to clone the repository. Make sure you are cloning **your**
    fork of this repo.

1.  Next, `cd` into the directory:

        $ cd the-name-of-directory-you-just-cloned

1.  At this point, you should be in your own local copy of the repository.

    As you work on the exercise below, be sure to frequently `commit` your work
    and `push` changes to the *remote* copy of the repo hosted on Github.
---

# <a name="variant-call-format"></a>
# Variant Call Formats

Variant call format (VCF) files are text-based genomic files with information on sequence variation. More specifically, they include sites where multiple characters are present in the samples examined. A VCF file contains a <b>[header section](#vcf-header-section)</b> and a <b>[variant data section](#vcf-data-section)</b>. Basic VCF files do not contain information on every position from the FASTQ or reference file, rather they include information on the genomic positions with sequence variation (although there is a different file type called a 'GVCF' (genomic variant call format) that includes information from all sites, see [here](https://gatk.broadinstitute.org/hc/en-us/articles/360035531812-GVCF-Genomic-Variant-Call-Format) for more info). As you probably gathered, a VCF is smaller than the FASTQ and SAM files (and the less variation, the smaller the file). Here is an abbreviated example of header and alignment lines within a VCF file:

##### abbreviated VCF file

```
##fileformat=VCFv4.2
##FORMAT=<ID=GT,Number=1,Type=String,Description="Genotype">
##FORMAT=<ID=AD,Number=R,Type=Integer,Description="Allelic depths for the ref and alt alleles in the order listed">
##FORMAT=<ID=DP,Number=1,Type=Integer,Description="Approximate read depth (reads with MQ=255 or with bad mates are filtered)">
##FORMAT=<ID=GQ,Number=1,Type=Integer,Description="Genotype Quality">
##INFO=<ID=AC,Number=A,Type=Integer,Description="Allele count in genotypes, for each ALT allele, in the same order as listed">
##INFO=<ID=AF,Number=A,Type=Float,Description="Allele Frequency, for each ALT allele, in the same order as listed">
##INFO=<ID=DP,Number=1,Type=Integer,Description="Approximate read depth; some reads may have been filtered">
##INFO=<ID=FS,Number=1,Type=Float,Description="Phred-scaled p-value using Fisher's exact test to detect strand bias">
##contig=<ID=NC_045541.1,length=186725308,assembly=reference.fasta>
##reference=file://reference.fasta
#CHROM  POS ID  REF ALT QUAL    FILTER  INFO    FORMAT  Sample01   Sample02   Sample03
NC_045541.1 1206    .   A   G   138.21  .   AC=2;AF=0.25;DP=6;FS=0.000 GT:AD:DP:GQ  0/0:6,0:6:42   ./.:0,0:0:0   1/0:5,7:12:71
```

# <a name="vcf-header-section"></a>
#### 1) VCF Header Section
The header section precedes the variant data section, and each heading begins with '##' symbols (notice there are *two* hash marks).  Information about the variant dataset, the reference sequence, and the program used to generate the VCF are contained within the header.
The ```FORMAT``` header lines define tags whose properties pertain to the variant site as a whole, whereas the ```INFO``` header lines describe tags whose properties pertain to the genotype for each individual in the dataset. The abbreviations in ```FORMAT``` and ```INFO``` header lines correspond with those in the data section. The abbreviated VCF file example above defines four ```FORMAT``` tags (GT, AD, DP, and GQ) and four ```INFO``` (AC, AF, DP, and FS) tags.  The ```contig``` and ```reference``` sections contain information about the reference used for variant calling.

# <a name="vcf-data-section"></a>
#### 1) VCF Variant Data Section
The variant section consists of a row for every variant. The columns provide information about (1) site-level properties and annotations and (2) sample-leve annotations. The first section of columns (site-level properties and annotations) correspond to the variant site as a whole. This section consists of 8 columns, all of which are required in the vcf file (```CHROM```, ```POS```, ```ID```, ```REF```, ```ALT```, ```QUAL```, ```FILTER```, ```INFO```). These required fields include information about the location, the reference allele, the alternate allele(s), and the quality of the SNP. While these fields are required for each variant, they can be empty (a signified by `.`). Descriptions for each of these fields are shown in the table below (values are from the variant of the abbreviated vcf file above).

| Col |  Field  |  Description                                                                   |  Value        |
|:---:|:-------:|:------------------------------------------------------------------------------:|:-------------:|
|  1  | ```CHROM```  |  Contig name                                                                   |  NC_045541.1  |
|  2  | ```POS```    |  Position of variant within contig                                             |  1206         |
|  3  | ```ID```     |  Optional identifier for variant                                               |  .            |
|  4  | ```REF```    |  Reference allele (sequence character(s) at POS in reference)                  |  A            |
|  5  | ```ALT```    |  Alternate allele (sequence character in at POS in at least one sample)        |  G            |
|  6  | ```QUAL```   |  Phred-scaled probability that variant exists at this site given data*         |  138.21       |
|  7  | ```FILTER``` |  ```PASS``` means the variant has passed filtering, . means no filtering has occurred|  .      |
|  8  | ```INFO```   |  Site-level annotations (properties of variant site as a whole)                |  AC=2;AF=0.25;DP=6;FS=0.000 |

###### \* Value of 10 means 0.1 chance of error; value of 100 means 0.0000000001 chance of error

The ```INFO``` values correspond to the flags defined in the header, where descriptions are provided. In the abbreviated vcf file, we see that this variant as an allele count (```AC```) of 2 (there are two alleles at this site), a minor allele frequency (```AF```) of 0.4 (the alternate allele's frequency in the dataset), a depth of coverate (```DP```) of 6 (the average depth per-individual at this site is 6 reads), and a p-value (```FS```) of 0.000. There are typically more property fields than this in a vcf, but this hopefully gives you a sense of how to read these sections.

The subsequent columns pertain to sample-level annotations. These fields consist of the formatting for the sample-specific property (```FORMAT```) followed by a column for each sample. In the abbreviated vcf file above, there are three samples (```Sample01```, ```Sample02```, and ```Sample03```). The values within the sample columns correspond to the ordered flags shown in the ```FORMAT``` column. These properties are shown in table format below; the Col values are a continuation from the table above.

| Col |  Field          |  Description                            | Value           |
|:---:|:---------------:|:---------------------------------------:|:---------------:|
|  9  |  ```FORMAT```   | Format for sample-specific annotations  | GT:AD:DP:GQ     |
|  10 |  ```Sample01``` | The annotation values for Sample 01     | 0/0:6,0:6:42    |
|  11 |  ```Sample02``` | The annotation values for Sample 02     | ./.:0,0:0:0     |
|  12 |  ```Sample03``` | The annotation values for Sample 03     | 1/0:5,7:12:71   |

The value column can be somewhat challenging to understand, so we'll break it down. The symbols in the ```FORMAT''' section are called "Genotype fields"

| Flag | Description                  | ```Sample01``` | ```Sample02``` | ```Sample03``` |
|:----:|:----------------------------:|:--------------:|:--------------:|:--------------:|
| GT   | Genotype\*                   | 0/0            | ./.            | 1/0            |
| AD   | Allele depth\*\*             | 6,0            | 0,0            | 5,7            |
| DP   | Total depth at variant site  | 6              | 0              | 12             |
| GQ   | Genotype quality\*\*\*       | 42             | 0              | 71             |

###### \* 0/0 = homozygous for ref allele; 1/1 = homozygous for alt allele; 1/0 = heterozygous; ./. no data
###### \*\* Depth for ref , depth for alt
###### \*\*\* Value of 10 means 0.1 chance of error; value of 100 means 0.0000000001 chance of error

# <a name="vcf-example"></a>
#### Looking at a VCF file

Now check out the example.vcf file. These files can be very large, but example.vcf is a small file that can be opened in your text editor. If on the command line, you can examine this file using ```vim example.vcf```.

> note: If you are new to using vim, you can remove text wrap by typing ':set nowrap' followed by enter. You can see line numbers by typing ':set number' followed by enter. You can exit vim without saving by typing ':q!' followed by enter.

You may notice that the ```FORMAT``` section contains more info than the example above (```AO```, ```GL```, ```GT```, ```QA```, ```RO```). There are also other genotype fields that you can include/exclude (see [this page](https://gatk.broadinstitute.org/hc/en-us/articles/360035531692-VCF-Variant-Call-Format) and  page 5 of [this document](http://samtools.github.io/hts-specs/VCFv4.1.pdf) for details of the other fields).

Once you have exited the text editor, you can count the number of variants in the VCF from the command line using the following command:

```
grep -v '#' example.vcf | wc -l
```

There are 1597 variants contained within this VCF file.

---

# <a name="variant-filtration"></a>
# Variant Filtration

#### Why filter variants?
When you call variants from an alignment file, you identify all sites where a read has a base call that is different than the reference. This includes sites with low coverage (a.k.a. read depth), sites with low allele depth (e.g., the read depth may be sufficient, but only one read varies from the reference), sites you are uninterested in (i.e., if you are focused on target loci you may not want to include all the called variants), and sites with low genotype quality. If left unfiltered, you risk including large amounts of false positives within your dataset. During downstream analysis these false positives may mask true biological patterns, therefore it is critical to remove them through a process known as variant filtration.

There are different strategies of variant filtration. For model organisms with known sites of variation, it is possible to perform [variant quality score recalibration](https://gatk.broadinstitute.org/hc/en-us/articles/360035531612-Variant-Quality-Score-Recalibration-VQSR) (VQSR), a machine learning approach that uses a training dataset of known variants

to recalibrate the variants. This is not possible in many non-model organisms without a dataset of known variants. An additional approach is sometimes called ["hard filtering"](https://gatk.broadinstitute.org/hc/en-us/articles/360035890471-Hard-filtering-germline-short-variants), which is where you decide on cutoff values for the genotype fields within a vcf. In this overview we focus on hard filtering.

Before filtering, you need to index your reference. This can be done using samtools and picard (I've already created index [fai] and dictionary [dict] files for the reference for the reference in this repository).

Multiple software packages can facilitate hard filtering of a VCF. These include [GATK](https://gatk.broadinstitute.org/hc/en-us/articles/360037434691-VariantFiltration), [vcftools](https://vcftools.sourceforge.net/man_latest.html), and [bcftools](https://samtools.github.io/bcftools/bcftools.html). With these packages you can do things like the following:

Filter variants based on their quality of depth (which incorporates variant confidence and raw depth):
```
gatk --java-options "-Xmx16g" VariantFiltration \
        -R  covid19-refseq.fasta \
        -V merged.vcf \
        -O qd_marked.vcf \
        --filter-name "QD" \
        --filter-expression "QD < 2.0"
```

For sites that pass this filtering criteria, they will be marked with ```PASS``` in the ```FILTER``` section of the VCF. If they don't pass the filtering criteria, they will be marked with ```QD``` in the ```FILTER``` section of the VCF. You can add multiple filtering criteria within this gatk command:

```
gatk --java-options "-Xmx16g" VariantFiltration \
        -R  covid19-refseq.fasta \
        -V merged.vcf \
        -O qd_marked.vcf \
        --filter-name "QD" \
        --filter-expression "QD < 2.0" \
        --filter-name "SOR" \
        --filter-expression "SOR > 3.0" \
        --filter-name "FS" \
        --filter-expression "FS > 60.0"
```

To remove sites that didn't pass your filtering criteria, you can use an awk command to remove anything that doesn't have ```PASS``` in the ```FILTER``` section of the vcf.

```
awk '/^#/||$7=="PASS"' qd_marked.vcf > qd_filtered.vcf
```

Using vcftools and bcftools, you can do many types of filtration:
```
# remove low-quality genotyping
vcftools --vcf qd_filtered.vcf --out gq_filtered --minDP 10 --recode --recode-INFO-all
# remove individuals from sites where they have low depth
vcftools --vcf gq_filtered.recode.vcf --out indDepth_filtered --minDP 10 --recode --recode-INFO-all
# remove sites that have more than 2 alleles
bcftools view -m2 -M2 -v snps indDepth_filtered.recode.vcf > multiallelic_filtered.vcf
# remove singletons (sites where only one individual has the alternate allele)
vcftools --vcf multiallelic_filtered.vcf --out singletons_filtered --mac 2 --recode --recode-INFO-all
# rename final file
mv singletons_filtered.recode.vcf filtered.vcf
```


# <a name="exercise"></a>
# Exercise
The data for this exercise come from [Farkas et al., 2021](https://doi.org/10.3389/fmicb.2021.665041) and the associated [github repository](https://github.com/cfarkas/SARS-CoV-2-freebayes).

The ```merged.vcf``` file is the output file from the [genomics-pipeline-intro](https://github.com/KLab-UT/genomics-pipeline-intro) repository.

## Exercise Objective
1. Modify the ```genomics-pipeline-intro.sh``` file so that it is compatible with the CHPC. In other words, add lines of code that will enable this script to run as a batch script so that the following command would work on the CHPC:

    ```
    sbatch bash_scripts/genomics-pipeline-intro.sh -l July_28_2020_NorAm.txt -g covid19-refseq.fasta -a 0.4999 -t 4
    ```

You don't need to run the full script- you already have the output (```merged.vcf```) here in this repository. You only need to modify the script so that it could be run on the CHPC.

2. Perform the filtering steps specified in the walkthrough above (using GATK, vcftools, and bcftools).

Once you have completed the worksheet, add, commit, and push the worksheet and the logfile to your forked repository.

```
add worksheet.md logfile
git commit -m "ran script and answered worksheet questions"
git push
```
