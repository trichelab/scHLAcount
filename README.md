# scHLAcount
Count molecules in single-cell RNA-seq data for class I genes HLA-A, B, and C; and class II genes DPA1, DPB1, DRA1, DRB1, DQA1, and DQB1 using a personalized reference genome.

```
scHLAcount DEV
Charlotte Darby <cdarby@jhu.edu> and Ian Fiddes <ian.fiddes@10xgenomics.com> and Patrick Marks <patrick@10xgenomics.com>
HLA genotyping and allele-specific expression for single-cell RNA sequencing

USAGE:
    sc_hla_count [FLAGS] [OPTIONS] --bam <FILE> --cell-barcodes <FILE>

FLAGS:
    -h, --help                  Prints help information
        --primary-alignments    If specified, will use primary alignments only
        --unmapped              If specified, will also use unmapped reads for genotyping (very slow!)
    -V, --version               Prints version information

OPTIONS:
    -b, --bam <FILE>                    Cellranger BAM file
    -c, --cell-barcodes <FILE>          File with cell barcodes to be evaluated
    -f, --fasta-cds <FILE>              Multi-FASTA file with CDS sequence of each allele [default: ]
    -g, --fasta-genomic <FILE>          Multi-FASTA file with genomic sequence of each allele [default: ]
    -d, --hladb-dir <PATH>              Directory of the IMGT-HLA database [default: ]
    -i, --hla-index <FILE>              debruijn_mapping pseudoalignment index file constructed from IMGT-HLA database [default: ]
        --log-level <log_level>         Logging level [default: error]  [possible values: info, debug, error]
    -o, --out-dir <OUTPUT_DIR>           [default: hla-typer-results]
        --pl-tmp <PSEUDOALIGNER_TMP>    Directory to write the pseudoaligner temporary files generated [default: .]
    -r, --region <STRING>               Samtools-format region string of reads to use [default: 6:28510120-33480577]
```  

# Case 1: You have HLA genotypes for some or all class I / class II genes

Other Requirements: samtools  

1. Download the the IMGT/HLA database, available at [Github](https://github.com/ANHIG/IMGTHLA) or [FTP](ftp://ftp.ebi.ac.uk/pub/databases/ipd/imgt/hla/). You only need the `hla_gen.fasta` and `hla_nuc.fasta` files, but you can download the whole database if you choose.
2. Use `samtools faidx` to index the `hla_gen.fasta` and `hla_nuc.fasta` files. 
3. Create a file of the known genotypes, at most two per gene, with one genotype on each line. Follow the template at `paper/sample_gt.txt`. 
4. We strongly recommend that if genotypes are unknown for any of the genes, you put the reference genome allele for those genes in the known genotypes file. Alleles represented in the GRCh38 primary assembly are listed below:
```
A*03:01:01:01
B*07:02:01:01
C*07:02:01:01
DQA1*01:02:01:01
DQB1*06:02:01:01
DRB1*15:01:01:01
DPA1*01:03:01:01
DPB1*04:01:01:01
```
5. If the indexed IMGT/HLA database files are not in the current directory, edit the `prepare_reference.sh` file to point to these files.
6. Run `prepare_reference.sh known_genotypes.txt` to get your custom references `cds.fasta` and `gen.fasta`. The samtools command will fail if the coding and genomic sequence of all alleles specified are not present in the database! If multiple alleles are present that match the provided level of specificity of the genotype, one will be chosen arbitrarily.
7. Run scHLAcount with the custom references as `-f` and `-g` parameters. Do not use the `-i` and `-d` parameters.


# Case 2: You do not have HLA genotypes

1. Download the the IMGT/HLA database, available at [Github](https://github.com/ANHIG/IMGTHLA) or [FTP](ftp://ftp.ebi.ac.uk/pub/databases/ipd/imgt/hla/). You only need the files `hla_gen.fasta`, `hla_nuc.fasta`, and `Allele_status.txt` but you can download the whole database if you choose.
2. The directory containing these files should be provided as the `-d` parameter to scHLAcount.
3. Run scHLAcount with the `-d` parameter. Do not use the `-f`, `-g`, or `-i` parameters.
4. If you run the program again and want to skip building the index, just specify the file `hla_nuc.fasta.idx` as the `-i` parameter. This file is located in the pseudoaligner temporary folder specified by the `--pl-tmp` parameter.
5. If you run the program again on and want to skip calculating the genotypes, you can use the `pseudoaligner_nuc.fa` and `pseudoaligner_gen.fa` as the `-f` and `-g` parameters. These files are located in the pseudoaligner temporary folder specified by the `--pl-tmp` parameter.