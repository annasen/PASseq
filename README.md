# PAS-seq

This workflow combines bulk **PAS-seq2 and CEL-Seq2**.
> Yoon Y, Soles LV, Shi Y. PAS-seq 2: A fast and sensitive method for global profiling of polyadenylated RNAs. Methods Enzymol. 2021;655:25-35. doi: 10.1016/bs.mie.2021.03.013. Epub 2021 Apr 23. PMID: 34183125. Hashimshony, T., Senderovich, N., Avital, G. et al.
> 
> CEL-Seq2: sensitive highly-multiplexed single-cell RNA-Seq. Genome Biol 17, 77 (2016). https://doi.org/10.1186/s13059-016-0938-8


### 1 Demultiplex pooled fastq data
During the library preparation, in order to save reagents and time, the samples were pooled after RT reaction since each sample was already indexed by the CEL-Seq2 poly-dT primer. This step, however, requires demultiplexing on our own as bcl2fastq step takes into account only samples sorted upon the second PCR indexed primer.

![R2_like-CELseq2](https://github.com/user-attachments/assets/aaac2b9e-b857-4bfe-9c9a-67c74a6536de)

I used the demultiplex package in my bash_env, I listed first R2 file as that is the place where the barcodes are stored (see in the picture, 8nt UMI, 6nt cell barcode, polyT). https://demultiplex.readthedocs.io/en/latest/usage.html  
Depending on the original fastq.gz size, the command can take some time. I recommend using screen command.  
There might be quite a big file sample_R*_UNKNOWN.fastq.gz.
```
demultiplex demux -r -s 9 -e 14 barcodes.tsv sample_R2.fastq.gz sample_R1.fastq.gz
```

The demultiplexed R1 sequences can be copied into a new folder because we will be using only R1 reads (as shown in the picture above, R2 reads contain only few nucleotides of the actual sequence at the very end, so we will not use them anymore, their main role was about having the cell barcode for demultiplexing). In order to get rid off the "sample_R1_" at the beginning of each (for clarity and for seq2science to run the pipeline as single-end), one can use this bash loop below:
```
for file in *fastq.gz; do mv "$file" "${file/sample_R1_/}"; done
```

Note: In shallow seq low RNA-quality experiments, the sorted fastq files can still contain a lot of R2 reads without the pattern -8 UMI, 6 barcode, polyT- (check with zcat sample_R2.fastq.gz | head -n 40). One can apply more stringent filtering by adding TTTTT at the end of each cell barcode sequence in the barcode_CEL-Seq2_48.tab file. 

### 2 Mapping by seq2science
Link to seq2science documentation https://vanheeringen-lab.github.io/seq2science/index.html   
Download annotation/index for mapping. Current version is 107 (2022/08/16) (https://github.com/erhard-lab/gedi/wiki/Preparing-genomes)
```
gedi -e IndexGenome -organism homo_sapiens -version 107 -p
```

seq2science to be run as single-end on the R1 samples.

Another thing to adjust in the config file might be removing the duplicates:
```
remove_dups: true # keep duplicates (check dupRadar in the MultiQC) true if you want to remove dupl
```
