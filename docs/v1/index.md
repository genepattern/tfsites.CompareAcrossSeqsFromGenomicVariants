# tfsites.CompareAcrossSeqsFromGenomicVariants v1

**Author(s):** Joe Solvason  

**Contact:** Joe Solvason (solvason@eng.ucsd.edu)

**Adapted as a GenePattern Module by:** Ted Liefeld (jliefeld@cloud.ucsd.edu)

**Task Type:** Transcription factor analysis

**LSID:**  urn:lsid:genepattern.org:module.analysis:00479


## Introduction

`tfsites.CompareAcrossSeqsFromGenomicVariants` tool fills this need by taking in a multiple sequence alignment of two or more enhancers to map how sequence variation impacts function of TF binding sites. In biomedical applications, comparisons can be made between reference and alternate alleles that are associated with diseases or changes in gene expression. In biomedical applications, `tfsites.CompareTfSitesAcrossSequencess` can be used to determine which binding sites are lost, gained, or changed across genetic variants of enhancers. In evolutionary applications, `tfsites.CompareTfSitesAcrossSequencess` can be used to determine which binding sites are lost, gained, or changed within a particular clade of species.

 
## Methodology

Enhancers that will be compared are labeled with groups – wild-type, control and test. Control sequences have sequence variation but have the same function (ie, sequence changes do not alter enhancer activity). Test sequences have sequence variation and drive differential enhancer activity. When a control group is available, it is helpful as it allows you to see which binding sites that uniquely arise in the sequence variants that drive differential expression. Sites that arise in both control and test groups are not as interesting. If a wild-type is provided, the analysis will report how all binding sites relate to the wild-type.

The `scan_seqs` function scans every provided enhancer sequence for binding sites. There are 3 methods to predict binding sites: score-pwm, core-only and core-affinity. 

1.	Score-pwm calculates a binding score using a PWM, and predicts binding sites that are above a particular binding score threshold. Many PWMs can be inputted into a single analysis which enables high-throughput of PWMs which could potentially explain enhancer differential activity. 
2.	Core-only uses IUPAC nomenclature to define a binding site core (e.g. GGAW where W=A/T). This will report de novo and deletions.
3.	Core-affinity also uses IUPAC nomenclature to define abinding site core, but will also integrate affinity datasets (i.e. MITOMI, PBM). This enables reporting not just if the sequence variant drives de novo or deletion, but what the affinity of each site is as well. In addition, this method will report sequence variants that drive substantial changes in affinity.

A multiple sequence alignment of all enhancers is inputted into the tool. When predicting binding sites within a single sequence, the tool ignores ‘-‘ characters. For example, if a PWM is 8bp long, the tool test each contiguous 8-mer that contains only A/T/G or C. Any ‘-‘ characters are skipped until a total of 8bp of nt are reached. However, if a ‘-‘ resides within a kmer (for example, the 8mer G-AGGAAGT which is 9bp long with 8bp of real sequence), it allows the start position to start at either the G or the -. This allows the tool to map orthologous binding sites across sequences which contain indels. 

Finally, compare seqs collapses on binding sites that appear in the same location across multiple sequences. Binding sites with scores or affinities found to be altered across genetic variants are reported. Gain-of-function binding sites are either de novo (binding site in test sequence but not wild-type or controls) or increase (binding site is present in all sequences, but the test sequence score or affinity is increased). Loss-of-function binding sites are either deletion (binding site is absent test sequence but present in wild-type or controls) or decrease (binding site is present in all sequences, but the test sequence score or affinity is decreased).

## Parameters

<span style="color: red;">*</span> indicates required parameter

### Input and Outputs

- **enhancer DNA alignment table data (.tsv)**<span style="color: red;">*</span>
    - Tab-separated file containing at least two DNA sequences to be analyzed. We suggest only inputting the alignment +/- 15 bp from where the genetic variation of interest occurs. if this is a SNV, you would simply input the 30bp window of the alignment containing the variant. If this is a deletion, you would add 15 bp upstream of the first “-” and 15bp downstream of the last “-”. 
- **enhancer functional group table (.tsv)**<span style="color: red;">*</span>
    - Tab-separated file which labels each functional group. Functional group can be either wild-type, control, test, or na. If “na”, all enhancers associated with that label will be removed from the analysis. Binding sites are searched for within the “test” group that are not present within the “wild-type” or “control” group. At a minimum both of these requirements must be met, (1) at least one “control” or “wild-type” must be provided; (2) at least one “test” must be provided.

<span style="color: red;">*</span>**Either tf affinity information (.tsv) or motif input file (JASPAR format) or both must be provided.**

- **tf affinity information (.tsv)**
    -   File containing  all the information for the transcription factors being analyzed, including its name, binding site definition, desired color on the plot, any PBM relative affinity data, and any PFM relative score data. 
- **motif input file (JASPAR format)**
    - JASPAR formatted file with multiple motifs. These can be PFMs as counts or fractions, or PWMs.

### Other Parameters

- **analysis name**<span style="color: red;">*</span>
    - Name of the analysis. Used as the prefix of all output filenames.
- **hypothesis**<span style="color: red;">*</span>
    - In the genetic variant do you expect a gain of a site, loss of a site, or would you like to search for both? Default is "Both".
- **minimum binding change (float)**<span style="color: red;">*</span>
    - The minimum change of affinity or PWM binding score classify as “increase” or “decrease” in score or affinity. Default is 0.1.


## Input Files
 
1.  enhancer alignment table data (.tsv)
2.  enhancer functional group table (.tsv)
3.  tf affinity information (.tsv)
4.  motif input file (JASPAR format)     
       
## Output Files

- **tf affinity information (.tsv)**
    - An output report of the predicted altered binding sites. Each associated PWM or binding affinity data is provided for every sequence variant. HTML reports are separated into ablations (abl), decreases (dec), de novos (dnv) and increases (inc).
- **altered binding site table (.tsv)**
    - This table contains all binding sites with unique IDs (which match those in the HTML report). It also ranks the binding sites by how LOF or GOF they are. Each row is a predicted binding site for a genetic variant, which also contains the calculated binding scores or affinities. 
    
  
## Example Data

[Example input data is available on github](https://github.com/genepattern/tfsites.CompareTfSitesAcrossSequences/gpunit/data)
    
## References

    
## Version Comments

- **1.0.0** (2023-12-06): Initial draft of document scaffold.
