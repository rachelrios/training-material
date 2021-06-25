---
layout: tutorial_hands_on

title: Identification of allelic variants in SARS-CoV-2 from deep sequencing reads
level: Introductory
enable: false
zenodo_link: ''
questions:
- Which biological questions are addressed by the tutorial?
- Which bioinformatics techniques are important to know for this type of data?
objectives:
- The learning objectives are the goals of the tutorial
- They will be informed by your audience and will communicate to them and to yourself
  what you should focus on during the course
- They are single sentences describing what a learner should be able to do once they
  have completed the tutorial
- You can use Bloom's Taxonomy to write effective learning objectives
time_estimation: 3H
key_points:
- The take-home messages
- They will appear at the end of the tutorial
contributors:
- wm75
- bebatut

---


# Introduction
{:.no_toc}

Effectively monitoring global infectious disease crises, such as the COVID-19 pandemic, requirescapacity to generate and analyze large volumes of sequencing data in near real time. These data have proven essential for monitoring the emergence and spread of new variants, and for understanding theevolutionary dynamics of the virus.

Two sequencing platforms in combination with several established library preparation strategies are epredominantly used to generate SARS-CoV-2 sequence data. However, data alone do not equal knowledge: they need to be analyzed. The Galaxy community developed analysis workflows to support the **identification of allelic variants (AVs) in SARS-CoV-2 from deep sequencing reads**. 

These workflows allow one to identify allelic variants in SARS-CoV-2 genomes with variant allele frequencies ranging from 5% to 100% (i.e., they detect variants with intermediate frequencies as well.

In this tutorial we will see how to run these workflows for the different types of input data:

- Single end data derived from Illumina-based RNAseq experiments
- Paired end data derived from Illumina-based RNAseq experiments
- Paired-end data generated with Illumina-based Ampliconic (ARTIC) protocols
- ONT fastq files generated with Oxford nanopore (ONT)-based Ampliconic (ARTIC) protocols

> ### Agenda
>
> In this tutorial, we will cover:
>
> 1. TOC
> {:toc}
>
{: .agenda}

# Prepare Galaxy and data

Any analysis should get their own Galaxy history. So let's start by creating a new one:

> ### {% icon hands_on %} Hands-on: Prepare the Galaxy history
>
> 1. Create a new history for this analysis
>
>    {% snippet faqs/galaxy/histories_create_new.md %}
>
> 2. Rename the history
>
>    {% snippet faqs/galaxy/histories_rename.md %}
>
{: .hands_on}

## Import auxiliary datasets

For extracting the variants, we need to get the SARS-CoV-2 reference (a sequence-identical copy of [NC_045512.2](https://www.ncbi.nlm.nih.gov/nuccore/NC_045512.2?report=fasta)) and gene name aliases (to give gene regions easily recognizable names), i.e. a custom tabular file mapping NCBI RefSeq Protein identifiers as used by snpEff version 4.5covid19 to their commonly used names. 
In addition, we need for datasets generated using ARTIC protocols 2 extra datasets: a BED file with ARTIC network primer scheme and a custom tabular file describing the amplicon grouping of the primers in the primer scheme file.

> ### {% icon comment %} Not using the standard ARTIC primer set for amplification?
> 
> If the samples for amplification were not prepared with the standard ARTIC primer set, custom files will need to be provided: the BED with the primer scheme and tha tabular file describing the amplicon grouping of the primers in the primer scheme file.
{: .comment}

> ### {% icon hands_on %} Hands-on: Import auxiliary datasets
>
> 1. Import the auxiliary datasets:
>    - the SARS-CoV-2 reference (`NC_045512.2_reference.fasta`)
>    - gene name aliases (`NC_045512.2_feature_mapping.tsv`)
>    - (if ARTIC protocol has been used) ARTIC network primer scheme (`ARTIC_nCoV-2019_v3.bed`)
>    - (if ARTIC protocol has been used) ARTIC amplicon grouping of the primers (`ARTIC_amplicon_info_v3.tsv`)
>
>    Several options are possible to import these datasets:
>
>    - Option 1: From the shared data library (in folder `GTN - Material` -> `Variant analysis` -> `Identification of allelic variants in SARS-CoV-2 from deep sequencing reads`)
>
>      {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
>
>    - Option 2: From [Zenodo]({{ page.zenodo_link }})
>
>      ```
>      {{ page.zenodo_link }}/NC_045512.2_feature_mapping.tsv
>      {{ page.zenodo_link }}/NC_045512.2_reference.fasta
>      {{ page.zenodo_link }}/ARTIC_nCoV-2019_v3.bed
>      {{ page.zenodo_link }}/ARTIC_amplicon_info_v3.tsv
>      ```
>
>      {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>    - Option 3: From a shared history:
>      - On [usegalaxy.org](https://usegalaxy.org/u/aun1/h/artic-v3)
>      - On [usegalaxy.eu](https://usegalaxy.eu/u/nekrut/h/artic-v3)
>      - On [usegalaxy.org.au](https://usegalaxy.org.au/u/nekrut/h/artic-v3)
>
>      {% snippet faqs/galaxy/histories_import.md %}
>
{: .hands_on}

## Get data

Before we can begin any Galaxy analysis, we need to upload the input data: FASTQ files with the sequenced viral RNA from different patients infected with SARS-COV-2. Several types of data are possible:

- Single end data derived from Illumina-based RNAseq experiments
- Paired end data derived from Illumina-based RNAseq experiments
- Paired-end data generated with Illumina-based Ampliconic (ARTIC) protocols
- ONT fastq files generated with Oxford nanopore (ONT)-based Ampliconic (ARTIC) protocols

We encourage you to use your own data there (with at least 2 samples). If you do not have any datasets available we provide some example datasets (paired-end data generated with Illumina-based Ampliconic (ARTIC) protocols) from [COG-UK](https://www.cogconsortium.uk/), the COVID-19 Genomics UK Consortium.

To upload the data, there are several possibilities depending of how many datasets and what their origin is:

- Import datasets (locally, on a given URL or from a shared data library) and organize them as a dataset colletion

  A dataset collection is a way to represent an arbitrarily large collection of samples as a singular entity within a user's workspace.

- Import from [Sequence Read Archive (SRA) at NCBI](https://www.ncbi.nlm.nih.gov/sra)

> ### {% icon hands_on %} Hands-on: Import datasets
>
> 1. Import the datasets
>
>    - Option 1: From your own local data using **Upload Data** (1-10 datasets)
>      
>      {% snippet faqs/galaxy/datasets_upload.md %}
>
>    - Option 2: From your own local data using **FTP** (>10 datasets)
>
>      {% snippet faqs/galaxy/datasets_upload_ftp.md %}
>
>    - Option 3: From the shared data library
>
>      {% snippet faqs/galaxy/datasets_import_from_data_library.md %}
>
>      *For our example datasets, the 28 `fastqsanger.gz` files in folder `GTN - Material` -> `Variant analysis` -> `Identification of allelic variants in SARS-CoV-2 from deep sequencing reads`*
>
>    - Option 4: From an external server with URL
>
>      {% snippet faqs/galaxy/datasets_import_via_link.md %}
>
>      For our example datasets, the datasets are stored on [Zenodo]({{ page.zenodo_link }}) and can be retrieved using the following URL:
>
>      ```
>      {{ page.zenodo_link }}/ERR5931005_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931005_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931006_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931006_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931007_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931007_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931008_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5931008_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949456_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949456_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949457_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949457_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949458_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949458_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949459_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949459_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949460_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949460_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949461_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949461_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949462_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949462_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949463_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949463_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949464_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949464_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949465_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949465_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949466_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949466_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949467_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949467_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949468_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949468_2.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949469_1.fastqsanger.gz
>      {{ page.zenodo_link }}/ERR5949469_2.fastqsanger.gz
>      ```
>
> 2. Create a collection to organize the data
>
>    - Option 1: Single-end data (Illumina or ONT data)
>
>      {% snippet faqs/galaxy/collections_build_list.md %}
>
>    - Option 2: Paired-end data (Illumina data)
>
>      {% snippet faqs/galaxy/collections_build_list_paired.md %}
>
>      *To do for example datasets: the example datasets are paired-end data with datasets with `_1` contain the forward reads and datasets with `_2` the reverse reads.*
>
{: .hands_on}

> ### {% icon comment %} Learning  Building collections automatically
>
> It is possible to build collections from tabular data containing URLs, sample sheets, list of accessions or identifiers, etc. [A dedicated tutorial is available to explain the different possibilities]({% link topics/galaxy-interface/tutorials/upload-rules/tutorial.md %}).
>
{: .comment}

Another possibility to get datasets is to import them directly from SRA.

> ### {% icon hands_on %} Hands-on: Import datasets from SRA
>
> 1. Create a new file with the SRA (or ENA) ids (one per line), e.g.:
> 
>    ```
>    ERR5931005
>    ERR5931006
>    ERR5931007
>    ERR5931008
>    ERR5949456
>    ERR5949459
>    ERR5949457
>    ERR5949458
>    ERR5949460
>    ERR5949461
>    ERR5949462
>    ERR5949463
>    ERR5949464
>    ERR5949465
>    ERR5949466
>    ERR5949467
>    ERR5949468
>    ERR5949469
>    ```
>    <!-- not working with these ids -->
>
>    {% snippet faqs/galaxy/datasets_create_new_file.md %}
>    
> 2. {% tool [Faster Download and Extract Reads in FASTQ](toolshed.g2.bx.psu.edu/repos/iuc/sra_tools/fasterq_dump/2.11.0+galaxy0) %} with the following parameters:
>     - *"select input type"*: `List of SRA accession, one per line`
>       - {% icon param-files %} *"sra accession list"*: created file
{: .hands_on}

# From FASTQ to annotated allelic variants

To identify the SARS-CoV-2 allelic variants (AVs), a first workflow convert the FASTQ files to annotated AV through a series of steps that include quality control, trimming, mapping, deduplication, AV calling, and filtering.

The tools and parameters used for these steps have been selected based on the input data and after extensive testing. 4 versions of the workflow are then available:

Workflow version 	| Input data | Read aligner | Variant caller
--- | --- | --- | ---
Illumina RNAseq SE | Single end data derived from RNAseq experiments | **bowtie2** | **lofreq**
Illumina RNAseq PE | Paired end data derived from RNAseq experiments | **bwa-mem** | **lofreq**
Illumina ARTIC | Paired-end data generated with Illumina-based Ampliconic (ARTIC) protocols | **bwa-mem** | **lofreq**
ONT ARTIC | ONT fastq files generated with Oxford nanopore (ONT)-based Ampliconic (ARTIC) protocols | **minimap2** | **medaka**

<!-- citations -->

> ### {% icon details %} About the workflows
>
> - The two Illumina RNASeq workflows (Illumina RNAseq SE and Illumina RNAseq PE) perform read mapping with **bwa-mem** and **bowtie2**, respectively, followed by sensitive allelic-variant (AV) calling across a wide range of AFs with **lofreq**
> - The workflow for Illumina-based ARTIC data (Illumina ARTIC) builds on the RNASeq workflow for paired-end data using the same steps for mapping (**bwa-mem**) and AV calling (**lofreq**), but adds extra logic operators for trimming ARTIC primer sequences off reads with the **ivar** package. In addition, this workflow uses **ivar** also to identify amplicons affected by ARTIC primer-binding site mutations and excludes reads derived from such “tainted” amplicons when calculating alternative allele frequences (AFs) of other AVs.
> - The workflow for ONT-sequenced ARTIC data (ONT ARTIC) is modeled after the alignment/AV-calling steps of the [ARTIC pipeline](https://artic.readthedocs.io/). It performs, essentially, the same steps as that pipeline’s minion command, i.e. read mapping with **minimap2** and AV calling with **medaka**. Like the Illumina ARTIC workflow it uses **ivar** for primer trimming. Since ONT-sequenced reads have a much higher error rate than Illumina-sequenced reads and are therefore plagued more by false-positive AV calls, this workflow makes no attempt to handle amplicons affected by potential primer-binding site mutations.
>
> All four workflows use **SnpEff**, specifically its 4.5covid19 version, for AV annotation.
>
> Workflows default to requiring an AF ≥ 0.05 and AV-supporting reads of ≥ 10 (these and all other parameters can be easily changed by the user). For an AV to be listed in the reports it must surpass these thresholds in at least one sample of the respective dataset. We estimate that for AV calls with an AF ≥ 0.05 our analyses have a false-positive rate of < 15% for both Illumina RNAseq and Illumina ARTIC data, while the true-positive rate of calling such low-frequency AVs is ~80% and approaches 100% for AVs with an AF ≥ 0.15. This estimate is based on an initial application of the Illumina RNAseq and Illumina ARTIC workflows to two samples for which data of both types had been obtained at the virology department of the University of Freiburg and the assumption that AVs supported by both sets of sequencing data are true AVs. The second threshold of 10 AV-supporting reads is applied to ensure that calculated AFs are sufficiently precise for all AVs.
>
{: .details}

> ### {% icon hands_on %} Hands-on: From FASTQ to annotated AVs
>
> 1. **Get the workflow** on Galaxy
>    
>    - Option 1: Use workflows available on [usegalaxy.org](https://usegalaxy.org/), [usegalaxy.eu](https://usegalaxy.eu/) or [usegalaxy.org.au](https://usegalaxy.org.au/)
>
>      Click on the appropriate workflow version (Illumina RNAseq PE for the example datasets)
>
>        &#8595; Type of data / Galaxy instance &#8594; | [usegalaxy.org](https://usegalaxy.org/) | [usegalaxy.eu](https://usegalaxy.eu/) | [usegalaxy.org.au](https://usegalaxy.org.au/)
>        --- | --- | --- | ---
>        Illumina ARTIC PE (*to use for example datasets*) | [{% icon workflow %}](https://usegalaxy.org/workflows/run?id=d4ea6cdd40522eb1) | [{% icon workflow %}](https://usegalaxy.eu/workflows/run?id=2f9fa06b1a927a07) | [{% icon workflow %}](https://usegalaxy.org.au/workflows/run?id=7fd58f5fc93f414e)
>        Illumina RNAseq PE | [{% icon workflow %}](https://usegalaxy.org/workflows/run?id=5a610d5a42d50cf3) | [{% icon workflow %}](https://usegalaxy.eu/workflows/run?id=e7ba6ca41d46baf2) | [{% icon workflow %}](https://usegalaxy.org.au/workflows/run?id=bd1cf22d47389742)
>        Illumina RNAseq SE | [{% icon workflow %}](https://usegalaxy.org/workflows/run?id=c092a3631d68ce38) | [{% icon workflow %}](https://usegalaxy.eu/workflows/run?id=6b41f7afbe14647d) | [{% icon workflow %}](https://usegalaxy.org.au/workflows/run?id=31dbd313e5c8160b)
>        ONT ARTIC | [{% icon workflow %}](https://usegalaxy.org/workflows/run?id=88d0b64011c3148c) | [{% icon workflow %}](https://usegalaxy.eu/workflows/run?id=17d7f9bddbc834f8) | [{% icon workflow %}](https://usegalaxy.org.au/workflows/run?id=0c6f7bfc826433c4)
>
>      The browser will open a new tab with Galaxy's workflow invocation interface.
>
>    - Option 2: Import the workflow
>
>      - Copy the URL (e.g. via right-click) of the appropriate workflow version or download it to your computer:
>        - [Illumina ARTIC PE](https://raw.githubusercontent.com/galaxyproject/iwc/main/workflows/sars-cov-2-variant-calling/sars-cov-2-pe-illumina-artic-variant-calling/pe-artic-variation.ga) (*to use for example datasets*)
>        - [Illumina RNAseq SE](https://raw.githubusercontent.com/galaxyproject/iwc/main/workflows/sars-cov-2-variant-calling/sars-cov-2-pe-illumina-artic-variant-calling/illumina_rnaseq_se.ga)
>        - [Illumina RNAseq PE](https://raw.githubusercontent.com/galaxyproject/iwc/main/workflows/sars-cov-2-variant-calling/sars-cov-2-pe-illumina-artic-variant-calling/illumina_rnaseq_pe.ga)
>        - [ONT ARTIC](https://raw.githubusercontent.com/galaxyproject/iwc/main/workflows/sars-cov-2-variant-calling/sars-cov-2-pe-illumina-artic-variant-calling/ont_artic.ga)
>        
>      - Import the workflow into Galaxy
>
>        {% snippet faqs/galaxy/workflows_import.md %}
>
> 2. Run **COVID-19: variation analysis on ...** {% icon workflow %} using the following parameters:
>
>    {% snippet faqs/galaxy/workflows_run.md %}
>
>    - *"Send results to a new history"*: `No`
>
>    - For **Illumina ARTIC PE** workflow (named **COVID-19: variation analysis on ARTIC PE data**),  *to use for example datasets*
>      - {% icon param-file %} *"1: ARTIC primers to amplicon assignments"*: `ARTIC_amplicon_info_v3.tsv` or `ARTIC amplicon info v3`
>      - {% icon param-file %} *"2: ARTIC primer BED"*: `ARTIC_nCoV-2019_v3.bed` or `ARTIC nCoV-2019 v3`
>      - {% icon param-file %} *"3: FASTA sequence of SARS-CoV-2"*: `NC_045512.2_reference.fasta` or `NC_045512.2 reference sequence`
>      - {% icon param-collection %} *"4: Paired Collection (fastqsanger) - A paired collection of fastq datasets to call variant from"*: paired collection created for the input datasets
>
>    - For **Illumina RNAseq PE** workflow (named **COVID-19: variation analysis on WGS PE data**)
>      - {% icon param-collection %} *"1: Paired Collection (fastqsanger)"*: paired collection created for the input datasets
>      - {% icon param-file %} *"2: NC_045512.2 FASTA sequence of SARS-CoV-2"*: `NC_045512.2_reference.fasta` or `NC_045512.2 reference sequence`
>
>    - For **Illumina RNAseq PE** workflow (named **COVID-19: variation analysis on WGS SE data**)
>      - {% icon param-collection %} *"1: Input dataset collection"*: dataset collection created for the input datasets
>      - {% icon param-file %} *"2: NC_045512.2 FASTA sequence of SARS-CoV-2"*: `NC_045512.2_reference.fasta` or `NC_045512.2 reference sequence`
>
>    - For **ONT ARTIC** workflow (named **COVID-19: variation analysis of ARTIC ONT data**)
>      - {% icon param-file %} *"1: ARTIC primer BED"*: `ARTIC_nCoV-2019_v3.bed` or `ARTIC nCoV-2019 v3`
>      - {% icon param-file %} *"2: FASTA sequence of SARS-CoV-2"*: `NC_045512.2_reference.fasta` or `NC_045512.2 reference sequence`
>      - {% icon param-collection %} *"3: Collection of ONT-sequenced reads"*: dataset collection created for the input datasets
{: .hands_on}

The execution of the workflow takes some time. It is possible to launch the next step even if it is not done, as long as all steps are successfully scheduled.

# From annotated AVs per sample to AV summary

Once the jobs of previous workflow are done, we identified AVs for each sample. We can run a "Reporting workflow" on them to generate a final AV summary. It  takes a table of AVs produced by any of the other four workflows and generates a list of AVs by Samples and by Variant. 

> ### {% icon hands_on %} Hands-on: From annotated AVs per sample to AV summary
>
> 1. **Get the workflow** on Galaxy
>    
>    - Option 1: Use workflow available on
>      - [usegalaxy.org](https://usegalaxy.org/workflows/run?id=2967ef82911f2cca)
>      - [usegalaxy.eu](https://usegalaxy.eu/workflows/run?id=0d10c137a0f08bca)
>      - [usegalaxy.org.au](https://usegalaxy.org.au/workflows/run?id=8e2b94c6fbf44368)
>
>    - Option 2: Import the workflow
>
>      - Copy the URL (e.g. via right-click) of [the workflow](https://raw.githubusercontent.com/galaxyproject/iwc/main/workflows/sars-cov-2-variant-calling/sars-cov-2-variation-reporting/variation-reporting.ga) or download it to your computer        
>      - Import the workflow into Galaxy
>
>        {% snippet faqs/galaxy/workflows_import.md %}
>
> 2. Run **COVID-19: variation analysis reporting** {% icon workflow %} using the following parameters:
>
>    {% snippet faqs/galaxy/workflows_run.md %}
>
>    - *"Send results to a new history"*: `No`
>    - *"1: AF Filter - Allele Frequency Filter"*: `0.05`
> 
>       This number is the nminimum allele frequency required for variants to be included in the report.
>
>    - *"2: DP Filer*: `1`
>
>       The minimum depth of all alignment at a variant site
>
>    - *"3: Variation data to report"*: `Final (SnpEff-) annotated variants`
>
>       The collection with variation data in VCF format: the output of the previous workflow
>
>    - *"4: gene products translations"*: `NC_045512.2_feature_mapping.tsv` or `NC_045512.2 feature mapping`
>
>       The custom tabular file mapping NCBI RefSeq Protein identifiers (as used by snpEff version 4.5covid19) to their commonly used names, part of the auxillary data.
>
{: .hands_on}

Both workflows generate several outputs. Most of them are collections with results for each samples. There are also reports combining information for all samples.

1. **Combined Variant Report by Sample**: table sumarize several information (columns) for each AVs in each sample (row)

   Column | Field | Meaning
   --- | --- | ---
   1 | `Sample` | SRA run ID
   2 | `POS` | Position in [NC_045512.2](https://www.ncbi.nlm.nih.gov/nuccore/1798174254)
   3 | `FILTER` | `Filter` field from VCF
   4 | `REF` |  Reference base
   5 | `ALT` | Alternative base
   6 | `DP` | Sequencing depth
   7 | `AF` | Alternative allele frequency
   8 | `SB` | Alternative allele frequency
   9 | `SB` | Strand bias P-value from Fisher's exact test calculated by [`lofreq`](https://csb5.github.io/lofreq/)
   10 |`DP4` | Depth for Forward Ref Counts, Reverse Ref Counts, Forward Alt Counts, Reverse Alt Counts
   11 |`IMPACT` | Functional impact (from SNPEff)
   12 |`FUNCLASS` | Funclass for change (from SNPEff)
   13 |`EFFECT` | Effect of change (from SNPEff)
   14 |`GENE` | Gene name
   15 |`CODON` | Codon
   16 |`AA` | Amino acid
   17 |`TRID` | Short name for the gene
   18 |`min(AF)` | Minimum Alternative Allele Freq across all samples containing this change
   19 |`max(AF)` | Maximum Alternative Allele Freq across all samples containing this change
   20 |`countunique(change)` | Number of distinct types of changes at this site across all samples
   21 |`countunique(FUNCLASS)` | Number of distinct FUNCLASS values at this site across all samples
   22 |`change` | Change at this site in this sample

   > ### {% icon question %} Questions
   > 
   > 1. How many AVs are found for all samples?
   > 2. How many AVs are found for the first sample in the document?
   > 3. How many AVs are found for each sample?
   >
   > > ### {% icon solution %} Solution
   > >
   > > 1. By expanding the dataset in the history, we have the number of lines in the file. 853 lines for the example datasets. The first line is the header of the table. Then 852 AVs.
   > >
   > > 2. We can filter the table to get only the AVs for the first sample {% tool [Filter data on any column using simple expressions](Filter1) %} with the following parameters:
   > >    - {% icon param-file %} *"Filter*": `Combined Variant Report by Variant`
   > >    - *"With following condition*": `c1=='ERR5931005'` (to adapt with the sample name)
   > >    - *"Number of header lines to skip*": `1`
   > >
   > >    We got then only the AVs for the selected sample (46 for ERR5931005).
   > >
   > > 3. To get the number of AVs for each sample, we can run {% tool [Group data](Grouping1) %} with the following parameters:
   > >    - {% icon param-file %} *"Select data"*: `Combined Variant Report by Variant`
   > >    - *"Group by column"*: `Column: 1`
   > >    - In *"Operation"*:
   > >      - In *"1: Operation"*:
   > >        - *"Type"*: `Count`
   > >        - *"On column"*: `Column: 2`
   > >    
   > >    With our example datasets, it seems that samples have between 41 and 56 AVs. 
   > {: .solution}
   {: .question}

2. **Combined Variant Report by Variant**: this table combine the information (column) for each AV (row)

   Column | Field | Meaning
   --- | --- | ---
   1 | `POS` | Position in [NC_045512.2](https://www.ncbi.nlm.nih.gov/nuccore/1798174254)
   2 | `REF` | Reference base
   3 | `ALT` | Alternative base
   4 | `IMPACT` | Functional impact (from SNPEff) 
   5 | `FUNCLASS` | Funclass for change (from SNPEff)
   6 | `EFFECT` | Effect of change (from SNPEff)
   7 | `GENE` | Gene 
   8 | `CODON` | Codon 
   9 | `AA` | Amino acid 
   10 |`TRID` | Short name for the gene 
   11 |`countunique(Sample)` | Number of distinct samples containing this change 
   12 |`min(AF)` | Minimum Alternative Allele Freq across all samples containing this change 
   13 |`max(AF)` | Maximum Alternative Allele Freq across all samples containing this change 
   14 |`SAMPLES(above-thresholds)` | List of distinct samples where this change has frequency abobe threshold (5%)
   15 |`SAMPLES(all)` | List of distinct samples containing this change at any frequency (including below threshold) 
   16 |`AFs(all)` | List of all allele frequencies across all samples 
   17 |`change` |  Change 

   > ### {% icon question %} Questions
   > 
   > 1. How many AVs are found?
   > 1. What are the different impacts of the AVs ?
   > 2. How many variants are found for each impact?
   > 3. What are the different effects for HIGH impact?
   > 4. Is there any AVs impacting all samples?
   >
   > > ### {% icon solution %} Solution
   > >
   > > 1. By expanding the dataset in the history, we have the number of lines in the file. 191 lines for the example datasets. The first line is the header of the table. Then 190 AVs.
   > >
   > > 2. The different impacts of the AVs are HIGH, MODERATE and LOW.
   > >
   > > 2. To get the number of AVs for each impact levels, we can run {% tool [Group data](Grouping1) %} with the following parameters:
   > >    - {% icon param-file %} *"Select data"*: `Combined Variant Report by Variant`
   > >    - *"Group by column"*: `Column: 4`
   > >    - In *"Operation"*:
   > >      - In *"1: Operation"*:
   > >        - *"Type"*: `Count`
   > >        - *"On column"*: `Column: 1`
   > >    
   > >    With our example datasets, we find:
   > >    - 18 AVs with no impact
   > >    - 51 LOW AVs
   > >    - 118 MODERATE AVs
   > >    - 9 HIGH AVs
   > >
   > > 3. We can filter the table to get only the AVs with HIGH impact by running {% tool [Filter data on any column using simple expressions](Filter1) %} with the following parameters:
   > >    - {% icon param-file %} *"Filter*": `Combined Variant Report by Variant`
   > >    - *"With following condition*": `c4=='HIGH'`
   > >    - *"Number of header lines to skip*": `1`
   > >
   > >    The different effects for the 9 HIGH AVs are STOP_GAINED and FRAME_SHIFT.
   > >
   > > 4. We can filter the table to get the AVs for which `countunique(Sample)` is equal the number of samples (18 in our example dataset): {% tool [Filter data on any column using simple expressions](Filter1) %} with the following parameters:
   > >    - {% icon param-file %} *"Filter*": `Combined Variant Report by Variant`
   > >    - *"With following condition*": `c11==18` (to adapt to the number of sample)
   > >    - *"Number of header lines to skip*": `1`
   > >
   > >    For our example datasets, 4 AVs are found for all samples
   > {: .solution}
   {: .question}

3. **Variant frequency**

   ![Variant frequency plot](../../images/sars-cov-2-variant-discovery/variant-frequency.svg)

   This plot represents AFs (cell color) for the different AVs (columns) and the different samples (rows). The AVs are grouped by genes (different colors on the 1st row). Information about their effect is also represented on the 2nd row. The samples are clustered following the tree displayed on the left.

   In the example datasets, the samples are clustered in 3 clusters (as we defined when running the workflow), that may represent different SARS-CoV-2 variants as the AVs profiles are different.

# From AVs to consensus sequences

> ### {% icon hands_on %} Hands-on: From AVs to consensus sequences
>
> 1. **Get the workflow** on Galaxy>
>      - Copy the URL (e.g. via right-click) of [the workflow](https://raw.githubusercontent.com/galaxyproject/iwc/main/workflows/sars-cov-2-variant-calling/sars-cov-2-consensus-from-variation/consensus-from-variation.ga) or download it to your computer        
>      - Import the workflow into Galaxy
>
>        {% snippet faqs/galaxy/workflows_import.md %}
>
> 2. Run **COVID-19: variation analysis reporting** {% icon workflow %} using the following parameters:
>
>    {% snippet faqs/galaxy/workflows_run.md %}
>
>    - *"Send results to a new history"*: `No`
>    - *"1: Variant calls"*: `Final (SnpEff-) annotated variants`
>
>       The collection with variation data in VCF format: the output of the first workflow
>
>    - *"2: min-AF for consensus variants*: `0.7`
>
>       Only variant calls with an AF greater than this value will be considered consensus variants.
>
>    - *"3: min-AF for failed variants"*: `0.25`
>
>       Variant calls with an AF higher than this value, but lower than the AF threshold for consensus variants will be considered questionable and the respective sites be masked (with Ns) in the consensus sequence.
>
>    - *"4: aligned reads data for depth calculation"*: `Full processed reads for variant calling`
>
>       Collection with fully processed BAMs generated by the first workflow. 
>
>       For ARTIC data, the BAMs should NOT have undergone processing with **ivar removereads**
>
>    - *"5: Depth-threshold for masking"*: `5`
>
>       Sites in the viral genome covered by less than this number of reads are considered questionable and will be masked (with Ns) in the consensus sequence independent of whether a variant has been called at them or not.
>
>    - *"6: Reference genome*": `NC_045512.2_reference.fasta` or `NC_045512.2 reference sequence`
>
>       SARS-CoV-2 reference genome, as part of the auxillary data.
>
{: .hands_on}


The main output of this worflow is the multisample consensus sequences in FASTA. This can be given as input for tools like **Pangolin** or **Nextclade**.

pangolin / nextclade

# Outlook

Worklow automation (tutorial)


# Conclusion
{:.no_toc}

Sum up the tutorial and the key takeaways here. We encourage adding an overview image of the
pipeline used.