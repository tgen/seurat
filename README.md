# seurat
Tumor-Normal Variant Caller

## NOTE
### Duplicate of Original Release at (https://sites.google.com/site/seuratsomatic/home/)

# Documentation
Seurat is a sequence analysis program for the discovery of biological events in paired tumor and normal genome and transcriptome data.

Release History
1.0 -       Initial release.

1.1 -       BAQ support, initial indel support.

2.0 -       Indel support, overlapping transcript support, structural variation support, new allelic imbalance detection algorithm, major                algorithm improvements, bug fixes, performance and output improvements.

2.1 -       Fully described insertions/deletions, updated default priors, bug fixes.

2.2 -       Severe SNV false-negative bug fixed. 

2.5 -       VCF formatting fixes, new '--metrics' option, usability & performance improvements.

 

Copyright
Seurat is Copyright (c) 2012 by The Translational Genomics Research Institute. All rights reserved. This License is limited to, and you may use the Software solely for, your own internal and non-commercial use for academic and research purposes. Without limiting the foregoing, you may not use the Software as part of, or in any way in connection with the production, marketing, sale or support of any commercial product or service or for any governmental purposes. For commercial or governmental use, please contact dcraig@tgen.org . By installing this Software you are agreeing to the terms of the LICENSE file distributed with this software.

 This software contains portions of the Genome Analysis Toolkit (GATK) which is Copyright (c) 2012, The Broad Institute and is used under license.

 

Usage
Seurat is a command-line Java application, packaged as a stand-alone JAR file. It is compatible with any operating system platform which is support by the Sun Java 1.6 runtime (including Linux, Windows, and Mac OS X).

Seurat can be executed using a command prompt (terminal) window of the operating system, by moving to the directory of the JAR file and executing the following command:

 

java -jar Seurat.jar -T Seurat -R (reference sequence FASTA file) -I:dna_normal (path to the indexed BAM of normal genome) -I:dna_tumor (path to the indexed BAM of tumor genome) -I:rna_normal (path to the indexed BAM of normal RNA BAM)] [-I:rna_tumor (path to the indexed BAM of tumor RNA BAM)] -o somatic_variants.vcf -go large_events.txt [ARGUMENTS...]

 

[ARGUMENTS...] are options picked from the section below.

                                                    

Supported arguments
Required:
 -I:dna_normal , -I:dna_tumor , -I:rna_normal , -I:dna_tumor    At least one normal/tumor input pair is needed for analysis. If RNA is provided, only RNA analysis will be performed (currently this is the allelic imbalance module).

-o <out>    Output VCF.

-go <gene_out>    Tab-delimited text output for non-focal events. Most large event analyses

require the 'refseq' argument below.

 

Optional:
--indels    Enable somatic insertion/deletion calling. Default = false.

--structvar    Enable structrual variation detection. Default = false.

-refseq <refseq_file>    Name of RefSeq transcript annotation file. If specified,

gene-wide events can be detected, and SNVs/LOH events will be annotated with the

gene name.

-Q <integer>    Minimum phred-scale for reported events. Default = 10.

-mbq <integer>    Minimum base quality required to consider a base for calling.

Default = 10.

-mmq <integer>    Minimum mapping quality for reads to be considered in the pileup.

Default = 10.

-ref <true/false>    If true, only reference-matching homozygous positions  are allowed on the normal, for SNV discovery. Reduces false positives due to faulty alignments. Default = true.

-alpha <integer>    Alpha parameter of the beta-distribution used for evaluating homozygosity likelihood. Default = 1.

-beta <integer> Beta parameter of the beta-distribution used for evaluating homozygosity likelihood. Default = 700.

--both_strands Whether or not variant evidence needs to appear on both strands on the tumor in order to be considered. Default = false.

-coding_only Reduces full-genome and transcript analyses to coding regions of genes. Requires the –refseq argument. Default = false.

-mm Maximum number of mismatches against the reference that are allowed in a read. Reads surpassing this number are filtered out. Can be used as an attempt to salvage ‘dirty’ BAMS containing large numbers of problematic and unlikely alignments, usually due to bugs in the aligner software.) Default =3.

-pileup Enable full pileup output for each call in the VCF file. Default = false.

-mcv <integer> The minimum per-sample coverage required to attempt a call at a locus. Default = 6.

--metrics Provide additional metrics for calls (eg. median base quality, median mapping quality, median cycle).

Usage notes / Known Issues
 Pre- and post-processing
We recommend the use of the GATK Indel Realigner to jointly process the normal and tumor DNA BAMs before analysis with Seurat, as we have empirically found that it reduces indel false positive counts significantly.
 We have found that the Base Quality recalibrator provides a minimal accuracy improvement.
We do not recommend the use of the base alignment quality (BAQ) (which can be enabled if needed with the ‘-baq’ argument). BAQ currently appears to be causing a significant drop in sensitivity.
 We do not recommend the use of the Variant call recalibrator on Seurat results, as the tool was not designed for somatic calls.
Additional functionality
Seurat accepts most global GATK arguments that can affect its functions. For more information on the GATK framework, please visit http://www.broadinstitute.org/gsa/wiki/index.php/The_Genome_Analysis_Toolkit.
Seurat does not support the ‘-nt’ option for running multiple threads within GATK.  However, the GATK interval option (“-L”) can be used to split the data into “bins” that can run simultaneously.
Empty results
If Seurat runs without any error messages and failures but the output files do not contain any calls, please check for the following conditions:
Read group tags (“@RG”) are required for all BAMs that are provided; BAMs without RG tags will be ignored (more accurately, any reads that are not assigned in a read group will not be used for analysis). If your BAMs were not generated with RG tags, you may use the Picard tool AddOrReplaceReadGroups to add them.  Please note (2) below if you have to add read groups manually.
The same sample name (“SM”) cannot be used on read groups belonging on both the normal and the tumor samples. GATK currently uses sample names to group alignments together, so if they are identical between datasets, they will be merged in-memory.
BAM files for analysis must match exactly on their header’s sequence names, sequence order, and sequence length.
For more information on how GATK handles BAM files, please refer to http://gatkforums.broadinstitute.org/discussion/1317/collected-faqs-about-bam-files
 

Example / Suggested use
 

java -jar Seurat.jar -T Seurat -R ref.fasta -I:dna_normal DNA_normal.BAM -I:dna_tumor DNA_tumor.BAM -I:rna_tumor RNA_tumor.BAM -o somatic_variants.vcf -go large_events.txt -Q 15 -refseq refseq.rod

 

Output formats
 

VCF (-o):
 

The text file follows the Variant Call Format (VCF) version 4.1, with one line per call. Please refer to the VCF format definition at http://www.1000genomes.org/wiki/Analysis/Variant%20Call%20Format/vcf-variant-call-format-version-41 for more information.

 

Variants detected will produce VCF records similar to the the following:

 

chr2 51001432 . G C 13.7 PASS TYPE=somatic_SNV;PILEUP=ggggGCGG/ccgGgcCgGcgGccCgg;DP=17

chr2 204596009 . T G 20.4 PASS TYPE=somatic_SNV;PILEUP=TttTtttttTttTtttttTTTttTtt/TTttttTtTttttttTTtTTTTTTTTTtGtTTtTtttgTGgGtt;DP=26

chr2    40279008    .   A   <DEL>   11.4    PASS    TYPE=somatic_deletion;PILEUP=aaaaAAAAAaaD/aADddddaaa;DP=10

 

 

The TYPE tag in the INFO field describes the somatic event that was detected.

The types currently are somatic_SNV, somatic_deletion, somatic_insertion and

LOH.

 

The ALT genotype describes either the variant detected in the tumor genome

(in case of a somatic SNV event), or the variant allele that is lost in

the tumor (in case of an LOH event). The strings "<INS>" and "<DEL>" represent

indels.

 

Large event list (-go):
 

A simple two-field tab-delimited text file in the following format:

 

[region]    [gene name] [event name/description]    [quality]   [additional info fields]

Comments
You do not have permission to add comments.
