# tfsites.CompareAcrossSeqsFromGenomicVariants v1

**Author(s):** Joe Solvason, Maggie Ma

**Contact:** Joe Solvason (solvason@ucsd.edu)

**Adapted as a GenePattern Module by:** Ted Liefeld (jliefeld@cloud.ucsd.edu)

**Task Type:** Transcription factor analysis

**LSID:**  urn:lsid:genepattern.org:module.analysis:00479


## Introduction

`tfsites.CompareAcrossSeqsFromGenomicVariants` takes in the genomic coordinates of variants to map how ref/alt sequence changes impact the function of TF binding sites. In biomedical applications, comparisons can be made between reference and alternate alleles that are associated with diseases or changes in gene expression. 

 
## Methodology

Reference and alternate sequences will be extracted based on the genomic coordinates provided. The size of the sequence extracted will depend on the window size, which is based on the length of the binding sites being analyzed. Each pair of ref/alt sequences will be analyzed separately. 

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

- **genome (.pkl)**<span style="color: red;">*</span>
    - Pickled genome file that corresponds to the genomic coordinates provided. This is used to extract the sequences to be compared.
- **variant file (.tsv)**<span style="color: red;">*</span>
    - Tab-separated file containing the list of genomic coordinates for the variants. 

<span style="color: red;">*</span>**Either tf affinity information (.tsv) or motif input file (JASPAR format) or both must be provided.**

- **tf affinity information (.tsv)**
    -   File containing  all the information for the transcription factors being analyzed, including its name, core site definition, and any affinity data (optional).
- **tf affinity files (.tsv)**
    - Affinity files referenced in tf affinity information. 
- **motif input file (JASPAR format)**
    - JASPAR formatted file with multiple motifs. These can be PFMs as counts or fractions, or PWMs. You can generate a PWM with the `tfsites.GenerateMotifDatabase` module.

### Other Parameters

- **analysis name**<span style="color: red;">*</span>
    - Name of the analysis. Used as the prefix of all output file names.
- **position index type (int)**<span style="color: red;">*</span>
    - Specify whether position coordinates are zero or one indexed.
- **window size (int)**<span style="color: red;">*</span>
    - 	Length of the binding sites that are being analyzed. This will be used to determine the number of nucleotides to include on each side of a variant when extracting the surrounding sequence.
- **minimum binding change (float)**
    - The minimum change of affinity or PWM binding score required to classify an “increase” or “decrease.” Default is `0.1`.
- **minimum pwm score (float)**
    - The minimum PWM binding score required to predict a site. Default is `0.8`.


## Input Files
 
1.  genome file (.pkl)
- Contains python dictionary object in the following format: {'chr1':'ACGTATTAGCCTAGAGATCA', ...}    
  
2.  variant file (.tsv)
- Assumes header is present
- Columns:
    - `Chr:` chromosome
    - `Pos:` position
    - `Ref:` reference allele
    - `Alt:` alternate allele
    - `Hypothesis`: specify whether gof/lof/both/na

```
Chr     Pos	        Ref	Alt	Hypothesis
chr2	65502802	G	A	gof
chr9	21776615	T	G	gof
chr9	21842300	G	C	lof
chr8	27337045	A	T	lof
```
 
3.  tf affinity information (.tsv)
- Assumes header is present
- Columns:
    - `TF Name:` name of the transcription factor
    - `Core Site:` minimal IUPAC binding site definition for transcription factor 
    - `Affinity Data (optional):` name of the relative affinity data file
 
```
TF Name    Core Site    Affinity Data
ETS        NNGGAWNN     input_ets1-pbm.tsv    
ETS-only   NNGGAWNN
```

4. tf affinity file (.tsv)

```
Kmer       Relative Affinity
AAAAAAAA   0.15
AAAAAAAC   0.11
AAAAAAAG   0.13
```
  
6.  motif input file (JASPAR format)
- Can provide multiple PWMs 

```
>MA1113.3	PBX2
A  [  4925  26620    225  24368  27245  27259    704   2298  25945 ]
C  [ 19645    629    588   2266    574    754    453  23894    848 ]
G  [  1585   1710    317    817    343    569    327    555    352 ]
T  [  3441    637  28466   2145   1434   1014  28112   2849   2451 ]
```
 
       
## Output Files

- **differential binding sites (.tsv)**
    - This table contains all binding sites with unique IDs (which match those in the HTML report). It also ranks the binding sites by how LOF or GOF they are. Each row is a predicted binding site for a genetic variant, which also contains the calculated binding scores or affinities. 
- **folders with html reports**
    - An output report of the predicted altered binding sites. Each associated PWM or binding affinity data is provided for every sequence variant. HTML reports are separated into ablations (abl), decreases (dec), de novos (dnv) and increases (inc).
    
  
## Example Data

[Example input data is available on github](https://github.com/genepattern/tfsites.CompareAcrossSeqsFromGenomicVariants/gpunit/data)
    
## References

    
## Version Comments

- **1.0.0** (2023-12-06): Initial draft of document scaffold.
