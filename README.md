# C3Q pipeline
## Cufflinks and CodingQuarry based gene prediction pipeline for intron rich fungal genomes

By Patrícia A. G. Ferrareze and Rodrigo S. A. Streit  




### What does C3Q pipeline do?  
The C3Q pipeline is a Cufflinks and CodingQuarry based gene prediction pipeline optimized for intron rich fungal genomes, as Cryptococcus species. The C3Q pipeline was developed and tested with C. neoformans H99 and C. deneoformans JEC21 genomes, using RNA-Seq as the primary source of information for the gene prediction. 
The C3Q pipeline was used to gene prediction in C. deuterogattii R265. The selection of parameters and results are described in the paper: 

**Application of an optimized annotation pipeline to the _Cryptococcus deuterogattii_ genome reveals dynamic primary metabolic gene clusters and genomic impact of RNAi loss**  
Patricia A. G. Ferrareze, Corinne Maufrais, Rodrigo Silva Araujo Streit, Shelby J. Priest, Christina A. Cuomo, Joseph Heitman, Charley C. Staats, Guilhem Janbon  
bioRxiv 2020.09.01.278374  
https://doi.org/10.1101/2020.09.01.278374  
<br />
### How does C3Q pipeline works?  
The C3Q pipeline performs the gene prediction using RNA-Seq alignment (.bam) and genome (.fna/.fa) files. The addition of a protein file of sequences from close species (.faa/.fa) is optional but recomended.  
The pipeline works as described below:  
1. The Cufflinks transcripts assembly (input: bam files from reads mapping - subsampled¹)  
2. The Cuffmerge combination of the assembled transcripts (input: the Cufflinks generated GTFs)  
3. The The CodingQuarry gene training and prediction with the merged assembled transcripts file (GFF) and the genome file (input: the Cuffmerge combined GFF file and the genome)  
4. The filtering of small dubious sequences (spliced sequences up to 150 nt, intronless sequences up to 300 nt) (input: the CodingQuarry gene prediction file)  
5. The HTSeq-count of reads for the predicted sequences and the filtering of genome-predicted sequences without reads  (The filtered CodingQuarry gene prediction file and the mapped BAM files)  
6. The filtering of alternative sequences from multitranscripts loci (selection of the best gene model) (input: the full filtered CodingQuarry gene prediction file)  
7. The recover of deleted and non-predicted loci with Exonerate mapping of orthologous genes (input: the multifiltered CodingQuarry gene prediction file and the fasta file of protein sequences from related species)  
<br />

¹In the paper we show that subsampled BAM alignments generate better results than the large whole files in the Cufflinks transcripts assembly. For our organism models, the optimal subsampled library size was of 7.5 million reads for each RNA-Seq replicate, but the optimal size may differ for other organisms. If you want to subsample your bam files before using the pipeline, see https://broadinstitute.github.io/picard/ for DownsampleSam tool installation and usage. Keep in mind that DownsampleSam subsamples libraries based on a probability, so the probability value required to generate a 7.5 million reads subsampling will depend on your original library size.  
<br />

### What does C3Q requires?
The C3Q pipeline was tested in Linux systems x64. Therefore, we do not guarantee it will work in other operational systems.  

The C3Q pipeline is built on Python and depends on Cufflinks suite (cufflinks, cuffmerge and gffread), gffcompare, HTseq-count and Exonerate.   
The tested versions of the dependencies and their repositories are listed bellow:  
- [Python v3.6.9](https://www.python.org/downloads/release/python-369/)  
- [Cufflinks v2.2.1](http://cole-trapnell-lab.github.io/cufflinks/)  
- [CodingQuarry v2.0](https://sourceforge.net/projects/codingquarry/)  
- [GFFcompare v0.10.1 and v0.10.6](https://github.com/gpertea/gffcompare)  
- [HTSeq-count of the 'HTSeq' framework, v0.12.4](https://htseq.readthedocs.io/en/master/index.html)  
- [Exonerate v2.3.0 with GFF3 support](https://github.com/hotdogee/exonerate-gff3)  

In the exception of Exonerate², all the dependencies are available in Bioconda channel for Conda-based installation.  
  
<br />

After installation of the required programs and dependencies, the C3Qpipeline should be allowed to be executed by using  

> chmod +x C3Qpipeline  

So it can be run in its directory as  

> ./C3Qpipeline  

One may also export the code directory to $PATH  

> export PATH=$PATH:/path/to/C3Qpipeline/directory/  

Or simply moving it to a bin directory that is already on your $PATH  

> mv C3Qpipeline /path/to/chosen/bin/  

So that it may be called from any directory as  

> C3Qpipeline  

<br />
²The Exonerate required by C3Q is a modified version that produces a different output format than the regular Exonerate, which is used by C3Q. Thus, any attempt of executing C3Q with a regular version of Exonerate such as the ones available on Bioconda will likely cause it to crash or even to produce inaccurate results.  
<br />  

<br />  


## C3Q PIPELINE USAGE										

<pre>
C3Qpipeline [OPTIONS]  

Required arguments:										
  -genome <file.fna>                    Genome file in FASTA format.						
  -libs <libs.txt>                      List of RNA-seq libraries as specified in READ ME³.			
  -strandness <yes/no/reverse>                Strandess of the RNA-seq library. Must be either "yes" (stranded), "no" (unstranded) or "reverse" (reversely stranded).								
												
Optional arguments:										
  --sublibs <sublibs.txt>                  List of sub-sampled libraries as specified in READ ME⁴.			
  --exo <file.faa>                      Protein fasta file for exonerate guidance.				
  --refine-exo                Sets exonerate to refine its alignments. This is very memory and time consuming.						
  --o <name>                        Output name.								
  --p <number>                        Number of cpu cores to be used. Default: 1				
  -h, --help                  Show this help message.							
</pre>

<br />

³A file containing the absolute paths of the BAM files from the reads alignment, one path per line. Those MUST not to be subsampled, as they are used by HTSeq to count mapped reads.   
⁴A file containing the absolute paths of the subsampled BAM files from the reads alignment, one path per line. Those libraries optimize Cufflinks transcripts assembly, although they are optional. In the absence of subsampled libraries, Cufflinks will use the same libraries provided for HTSeq. Check "How does C3Q pipeline works?" for subsampling instructions.  
<br />

Example structure for both -libs and --sublibs files:  
<br />
*/home/user/Documents/my_experiment/library_1.bam*  
*/home/user/Documents/my_experiment/library_2.bam*  
*/home/user/Documents/my_experiment/library_3.bam*  
<br />

#### Some usage examples:  

To perform the complete pipeline added to the option of refine Exonerate alignments (memory and time consuming):  

>C3Qpipeline -genome genome_file.fna -libs libs_path.txt -strandness yes --sublibs subsampled_libs_path.txt --exo protein_file.faa --refine-exo  


To perform the complete pipeline with reduced memory and time:  

>C3Qpipeline -genome genome_file.fna -libs libs_path.txt -strandness yes --sublibs subsampled_libs_path.txt --exo protein_file.faa  


To perform the pipeline with the full size BAM libraries (no subsampling):  

>C3Qpipeline -genome genome_file.fna -libs libs_path.txt -strandness yes --exo protein_file.faa --refine-exo  


To perform the pipeline without the mapping of related protein sequences by Exonerate (RNA-Seq and genome based prediction only):  

>C3Qpipeline -genome genome_file.fna -libs libs_path.txt -strandness yes --sublibs subsampled_libs_path.txt  

<br />  
<br />

If you use this pipeline, please cite the paper:  
**Application of an optimized annotation pipeline to the _Cryptococcus deuterogattii_ genome reveals dynamic primary metabolic gene clusters and genomic impact of RNAi loss**  
Patricia A. G. Ferrareze, Corinne Maufrais, Rodrigo Silva Araujo Streit, Shelby J. Priest, Christina A. Cuomo, Joseph Heitman, Charley C. Staats, Guilhem Janbon  
bioRxiv 2020.09.01.278374  
https://doi.org/10.1101/2020.09.01.278374  


