# BINF_toolkit
This directory consists of scripts developed by Yu Wan for routine bioinformatic analysis.  

## A list of scripts
* [add_sample_name_FASTA.py](#add_sample\_name\_FASTA)
* [download_NCBI_records.py](#download\_NCBI\_records)
* [extract_nucl_region.py](#extract\_nucl\_region)
* [gbk2tbl.py](#gbk2tbl)
* [gc.py](#gc)
* [get_gene_seq.py](#get\_gene\_seq)
* [parse_ENA_sampleInfo_XML.py](#parse\_ENA\_sampleInfo\_XML)
* [run_CutAdapt.py](#run_CutAdapt)

## Manual
### <a name="add_sample_name_FASTA"></a>add\_sample\_name\_FASTA.py

This script appends a sample name at the beginning of each sequence in a FASTA file. For example, the header "\>g1 description" becomes "\>sample1__g1 description" after running this script.  

Command example: python add_sample_name_FASTA.py -i filename.txt (or filename.fna) -o output_dir -n  

### <a name="download_NCBI_records"></a>download_NCBI_records.py

This script takes as input a list of NCBI accession numbers (one for each line) from the STDIN and downloads corresponding entries (either GenBank files or FASTA files) under the target directory.  

**Examples**  
python download\_NCBI\_records.py --records "file:objects.txt" --format fasta --email xxx@xxx.com --suffix fna --outdir ./ref --skip > download.log  

python download\_NCBI\_records.py --records "NC_0001,NC_0002" --format genbank --email xxx@xxx.com --suffix gbk --outdir ./ref --skip > download.log  

_Type python download_NCBI_records.py -h or --help for help information._  

**Notes about options and option arguments**
* --records: can be either a file (must contain a suffix of ".txt") listing targets to be downloaded, or a string of accession IDs separated by commas (no space is allowed).
* --format or -f: the format of files to be downloaded
* --suffix or -s: the file extension, can be "fasta" (default), "fna", "gb", or "gbk". No dot preceding the extension is needed.
* --outdir or -o: output directory, no backslash at the end.

An example of the input list: seq_list.txt. Note that accession IDs may not include version numbers such as ".1" (HG326223.1, CP011642).  

**References**  
1. This script is inspired by Mark Schultz's (dr.mark.schultz@gmail.com, GitHub: schultzm) script "downloadGenbankByAccessions.py".
2. [A post on the BioStars forum](www.biostars.org/p/63506/)

### <a name="extract_nucl_region"></a>extract\_nucl\_region.py

This script extracts a region of nucleotides by positions from a fasta file.  

**Arguments**  
-i: the path of the input file  
-n: the name of your selected contig  
-f: feature name specified by the user  
-s: the first nucleotide to be selected  
-e: the last nucelotide to be selected  
-o: the filename of the output  
	
**Requirements**
* Only one genomic region should be selected;
* the start and end positions should not spill out.

### <a name="gbk2tbl"></a>gbk2tbl.py

This script converts a GenBank file (.gbk or .gb) from Stdin into a Sequin feature table (.tbl), which is an input file of tbl2asn used for creating an ASN.1 file (.sqn).  

Package requirement: BioPython and argparse  

**Usage**   
python gbk2tbl.py --mincontigsize 200 --prefix any_prefix --modifiers modifier_file.txt \< annotation.gbk 2\> stderr.txt  

Note that this script reads the GenBank file through the stdin ("\< annotation.gbk") and you may want to redirect the stderr to a file via "\> stderr.txt" (redirection).  

**Inputs**  
A GenBank file, which ought to be passed to the script through the standard input (stdin).  

A modifier file: a plain text file containing modifiers for every FASTA definition line.  
* All modifiers must be written in a single line and are separated by a single space character.
* No space should be placed besides the '=' sign. Check [NCBI help](http://www.ncbi.nlm.nih.gov/Sequin/modifiers.html) for choosing a proper format for modifiers.
* For example: a line "[organism=Serratia marcescens subsp. marcescens] [sub-species=marcescens] [strain=AH0650_Sm1] [topology=linear] [moltype=DNA] [tech=wgs] [gcode=11] [country=Australia] [isolation-source=sputum]" will be copied and printed along with the record name as the definition line of every contig sequence.  

**Outputs**  
* any_prefix.tbl: the Sequin feature table
* any_prefix.fsa: the corresponding fasta file
These files are inputs for tbl2asn which generates ASN.1 files (*.sqn).  

**Arguments**  
* --mincontigsize: the minimum contig size, default = 200 in accordance with NCBI's regulation  
* --prefix: the prefix of output filenames, default = 'seq'  
* --modifiers: the filename of the modifier file, default = 'modifiers.txt'  
	
### <a name="gc"></a>gc.py

This program calculates the length, GC content, and entropy for each record in a multi-fasta file.  

Input: a fasta file which contains multiple sequences from the standard input  

Output: for each sequence, the script prints: the header line, total sequence length, (G+C)% and entropy of the input sequence.  

Command line: python gc.py \< filename.fasta  

Treatment of the extended alphabet in this script:
1. consider all of 15 characters
2. construct a weighted-count table using dictionary
3. for each character in the table, take the probability of being A, G, C or T as effective counts
4. counts for A, G, C and T is computed by adding up the vectors for every character read from the sequence.

### <a name="get_gene_seq"></a>get_gene_seq.py

This script extracts gene sequences from a GenBank file, in accordance with a list of (locus_tag, feature type) tuples.
Required module: Bio, argparse, csv  

**Usage**: python get\_gene\_seq.py --tags _locus\_tag file_ --gb _GenBank file_ \> genes.fna  

**Inputs**  
1. A GenBank file.  
2. A text file listing selected locus_tags in the following format: locus_tag"\t"feature_type. This file MUST use ASCII codes because [the module csv/2.3 does not support Unicode inputs](https://docs.python.org/2/library/csv.html).  
3. Allowed feature types are: CDS, tRNA, rRNA and tmRNA. For example:  
SMDB11_RS00910	rRNA<br/>
SMDB11_RS21915	rRNA<br/>
SMDB11_RS00015	CDS<br/>

**Output**
Nucleotide sequences in FASTA format with the header in the format: \>feature type|contig name|locus_tag|position|length|product

**Warnings**  
1. Although it is unlikely in a GenBank file, but please always ensure that there is no duplication of locus_tags in the table because this script treats locus_tag"s as keys for retrieving feature types.
2. An "IndexError: list index out of range" will arise if the tag list uses Unicode codes.

### <a name="parse_ENA_sampleInfo_XML"></a>parse_ENA_sampleInfo_XML.py

This script parses an ENA metadata file in XML format and prints a subset of information.  

**Usage**: python parse\_ENA\_sampleInfo\_XML.py ERP000909.xml > samples.txt

Input: an XML file exported for a list of ERS accession numbers from ENA using the REST URLs API. For example, one can download an XML file for sample ERS086023 using the link [http://www.ebi.ac.uk/ena/data/view/ERS086023&display=xml](http://www.ebi.ac.uk/ena/data/view/ERS086023&display=xml).

**Outputs**  
* tab-delimited text file containing information retrieved from the XML file.  
* study_accession, sample_accession, secondary_sample_accession, experiment_accession, run_accession, Isolate_ID, Host, Place_of_isolation, Year_of_isolation  

### <a name="run_CutAdapt"></a>run_CutAdapt.py

This script runs [CutAdapt](https://github.com/marcelm/cutadapt) for a list of paired-end readsets.  

Dependency: [slurm](http://slurm.schedmd.com) on a computational cluster (Linux OS)
