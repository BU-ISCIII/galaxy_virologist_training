# Analysis of viral genomes using the Galaxy platform
# **Exercise 3: Illumina Assembly**

<div class="tables-start"></div>

|**Title**| Galaxy |
|---------|-------------------------------------------|
|**Training dataset:**|  [**PRJEB43037**](https://www.ncbi.nlm.nih.gov/bioproject/PRJEB43037/) - In August 2020, an outbreak of **West Nile Virus** affected 71 people with meningoencephalitis in Andalusia and 6 more people in Extremadura (south-west of Spain), causing a total of eight deaths. The virus belonged to the lineage 1 and was relatively similar to previous outbreaks occurred in the Mediterranean region. Here, we present a detailed analysis of the outbreak, including an extensive phylogenetic study. This is one of the outbreak samples.
|**Questions:**| <ul><li>What is an assembly?</li><li>How can we evaluate our assembly?</li></ul>|
|**Objectives**:|<ul><li>Understand the assembly concept.</li><li>Learn how to interpret assembly quality control metrics.</li></ul>|
|**Estimated time**:| 40 min |

<div class="tables-end"></div>

## **1. Description**

Sometimes, we don't have a reference genome to map our reads against, or we want to reconstruct a genome without any possible bias caused by a reference. In such cases, we need to perform _***de novo* assembly**_. This type of analysis tries to reconstruct the original genome without any templates, using only the reads themselves. 
Some considerations to take into account beforehand:

- When we assemble, the longer the reads are and the longer the size of the library fragments, the easier it is for the assembler. That's why PacBio or Nanopore are recommended for assembly. Think of it like a puzzle: the bigger the pieces, the easier it is to form the image.
- It's almost imposible to reconstruct the entire genome of a large-genome microorganism with only one sequencing, although it can be done for smaller ones, like viruses.
- Assembly is not recommended for amplicon based libraries due to the depth of coverage unevenness and the amplicons' intrinsic bias.

## **2. Upload data to Galaxy**

### **Training dataset**

- **Experiment info**: [**PRJEB43037**](https://www.ncbi.nlm.nih.gov/bioproject/PRJEB43037/), WGS, Illumina iSeq 100, paired-end.
- **Fastq R1**: [**ERR5310322_1**](https://ftp.sra.ebi.ac.uk/vol1/fastq/ERR531/002/ERR5310322/ERR5310322_1.fastq.gz) - URL : `ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR531/002/ERR5310322/ERR5310322_1.fastq.gz`
- **Fastq R2**: [**ERR5310322_2**](https://ftp.sra.ebi.ac.uk/vol1/fastq/ERR531/002/ERR5310322/ERR5310322_2.fastq.gz)  URL : `ftp://ftp.sra.ebi.ac.uk/vol1/fastq/ERR531/002/ERR5310322/ERR5310322_2.fastq.gz`
- **Reference genome** **NC_009942.1**: [**FASTA**](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/875/385/GCF_000875385.1_ViralProj30293/GCF_000875385.1_ViralProj30293_genomic.fna.gz) -- [**GFF**](https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/875/385/GCF_000875385.1_ViralProj30293/GCF_000875385.1_ViralProj30293_genomic.gff.gz)

### **Create new history**

- Click on the `+` icon at the top of the history panel and create a new history with the name `Illumina Assembly` as explained [**here**](https://github.com/BU-ISCIII/galaxy_virologist_training/blob/one_week_format/exercises/01_introduction_to_galaxy.md#2-galaxys-history).

### **Upload data**

- Import the read files `ERR5310322_1.fastq.gz` and `ERR5310322_2.fastq.gz`
    1. Click on **Upload**.
    2. Click on **Paste/Fetch data**.
    3. Copy URL for FASTQ R1 (select and Ctrl+C) and paste (Ctrl+V).
    4. Click on **Start**.
    5. Wait until the job finishes (green in history).
    6. Do the same for FASTQ R2.

<p align="center"><img src="images/upload_data_mapping.png" alt="Upload data mapping" width="900"></p>

- You can then **add tags** (``#R1`` & ``#R2``) to your reads, as we did in the preprocessing exercise. 

- Import the **reference genome** and **GFF files** that will be used for assembly quality assessment at the end of the practice. You can add tags to them as well.

```bash
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/875/385/GCF_000875385.1_ViralProj30293/GCF_000875385.1_ViralProj30293_genomic.fna.gz
https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/875/385/GCF_000875385.1_ViralProj30293/GCF_000875385.1_ViralProj30293_genomic.gff.gz
```

<p align="center"><img src="images/upload_data_mapping2.png" alt="Upload data mapping 2" width="900"></p>

- You can rename the reference genome and GFF files if you like:
    1. Click on the ✏️ for the reference file in the history.
    2. Change the name to `NC_009942.1`.

<p align="center"><img src="images/changename2.png" alt="Change name 2" width="900"></p>

## **3. Assemble reads with SPAdes**

1. Search `SPAdes` in the search tool box and select _**rnaviralSPAdes - de novo assembler for transcriptomes, metatranscriptomes and metaviromes**_.
>[!Note]
>[**SPAdes**](https://github.com/ablab/spades) is an assembler thas has multiple modes depending on the nature of the reads to be used for assembly. For instance, it has:
>- metaSPAdes: assembly of metagenomic data containing sequences from multiple organisms.
>- rnaSPAdes: transcriptome assembly from RNA-seq data.
>- rnaviralSPAdes: assembly of RNA viral genomes from viral RNA-seq data.
>- plasmidSPAdes and metaplasmidSPAdes: recovery of plasmids from whole-genome and metagenomic datasets, respectively.
>- metaviralSPAdes: recovery of viral sequences from metagenomic data.
2. Single-end or paired-end short-reads > **Paired-end: list of dataset pairs**.

>[!WARNING]
To be able to continue, it is necessary to first create a **list of paired datasets**, given our FASTQ files, which will then be used for the rest of the steps of this workshop. In order to do so, we must:
>
> 1. **Select our fastqsanger.gz files that we uploaded in the beginning, and then click on _2 of X selected_**.
>
>    <p align="center"><img src="images/select_items.png" alt="select_items" width="900"></p>
>
> 2. **Select _Advanced Build List_**.
>
>    <p align="center"><img src="images/advanced_build_list.png" alt="advanced_build_list" width="900"></p>
>
> 3. **Select the _List of Paired Datasets_ box and click on *Next***.
>
>    <p align="center"><img src="images/list_of_paired_datasets.png" alt="list_of_paired_datasets" width="900"></p>
>
> 4. **Check the auto-pairing configuration and click again on *Next***.
>
>    <p align="center"><img src="images/configure_auto_pairing.png" alt="configure_auto_pairing" width="900"></p>
>
> 5. **Give a name to your list and finally click on *Build***.
>
>    <p align="center"><img src="images/list_identifier.png" alt="list_identifier" width="900"></p>
>
> **After having done this, a list of datasets will have been generated:**
>
>    <p align="center"><img src="images/list_of_datasets.png" alt="list_identifier" width="900"></p>
>    <p align="center"><img src="images/pair_of_datasets.png" alt="list_identifier" width="900"></p>
>
> **This list will have the forward and reverse reads:**
>
>    <p align="center"><img src="images/forward_reads.png" alt="list_identifier" width="900"></p>
>    <p align="center"><img src="images/reverse_reads.png" alt="list_identifier" width="900"></p>
>
> **Once this is done, you'll be able to select your list as input of SPAdes.**

1. FASTQ RNA-seq file(s): collection: **ERR5310322**
2. Select optional output file(s) > **Scaffolds stats** and **Contigs stats**, along with the options selected by default.
3. Click on **Run Tool** and wait.

<p align="center"><img src="images/spades_params.png" alt="Spades params" width="900"></p>
<p align="center"><img src="images/spades_params_2.png" alt="Spades params 2" width="900"></p>

>[!WARNING]
**Assembly takes time!**
> There is no such thing as Assembly in real time. This process might take long.

**Questions:**

Click on the **eye icon** in the history: **Spades Contigs stats**.
    <details>
    <summary>How many contigs have been assembled?</summary>
    <b>46</b>
    </details>

## **4. Assembly quality control with QUAST**

1. Search **QUAST** in the search tool box and select **Quast - Genome assembly Quality**.
2. **Assembly mode? > Individual assembly (1 contig file per sample)**.
3. Contigs/scaffolds file > **rnaviralSpades Scaffolds**
4. Use a reference genome? > **Yes**. Select the **FASTA file** previously loaded.
5. Genomic feature positions in the reference genome > **GFF file** previously loaded.

<p align="center"><img src="images/quast_params1.png" alt="quast params" width="900"></p>
<p align="center"><img src="images/quast_params2.png" alt="quast params" width="900"></p>

6. **Run Tool**.
7. Click on the eye icon of the **QUAST HTML report**.
    <details>
    <summary>How much of our reference genome have we reconstructed?</summary>
    <b>Genome fraction: 98.576%</b>
    </details>
    <details>
    <summary>How many contigs do we have greater than 1000 pb?</summary>
    <b>1</b>
    </details>
    <details>
    <summary>How long is the largest contig in the assembly?</summary>
    <b>11615 (only one contig)</b>
    </details>
    <details>
    <summary>Which is the N50?</summary>
    <b>11615</b>
    </details>

8. Open the **Icarus** viewer in the QUAST report.

<p align="center"><img src="images/icarus.png" alt="quast params" width="400"></p>

<details>
 <summary>How did the contig align against our reference genome?</summary>
 <b>Misassembled blocks</b>. The assembly consists of a single contig that covers about 98.6% of the reference genome, but QUAST flags it as misassembled because it reports one local misassembly (there is an overlap of < 1 kb). Here, two parts of the same contig align to overlapping reference positions, meaning a ~575 bp region (positions 1,937–2,511) appears twice in the contig.
</details>
<br>

> **This training history is available at: https://usegalaxy.eu/u/s.varona/h/illumina-assembly-101-tutorial**
