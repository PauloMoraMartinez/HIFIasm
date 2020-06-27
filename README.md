## Getting Started

```sh
# Install hifiasm (requiring g++ and zlib)
git clone https://github.com/chhylp123/hifiasm
cd hifiasm && make
# Assembly
./hifiasm -o NA12878.asm -t 32 NA12878.fq.gz
```

## Introduction

Hifiasm is a fast haplotype-resolved de novo assembler for PacBio
Hifi reads. Unlike most existing assemblers, hifiasm starts from uncollapsed
genome. Thus, it is able to keep the haplotype information as much as possible.

For non-trio assembly, the input of hifiasm is the PacBio Hifi reads in fasta/fastq format, and its
outputs consist of: 

1. Haplotype-resolved raw [unitig][unitig] graph in [GFA][gfa] format
   (*prefix*.r\_utg.gfa). This graph keeps all haplotype information, including
   somatic mutations and recurrent sequencing errors.
2. Haplotype-resolved processed unitig graph without small bubbles
   (*prefix*.p\_utg.gfa). Small bubbles might be caused by somatic mutations or noise in data, 
   which are not the real haplotype information.
3. Primary assembly [contig][unitig] graph (*prefix*.p\_ctg.gfa). This graph collapses different
   haplotypes.
4. Alternate assembly contig graph (*prefix*.a\_ctg.gfa). This graph consists of all assemblies that
   are discarded in primary contig graph.

For trio assembly, the input of hifiasm is the PacBio Hifi reads in fasta/fastq format, and the paternal/maternal trio indexes generated by `yak count` (see https://github.com/lh3/yak). The outputs consist of:
1. Haplotype-resolved raw [unitig][unitig] graph in [GFA][gfa] format
   (*prefix*.r\_utg.gfa). This graph keeps all haplotype information. 

2. Phased paternal/haplotype1 contig graph (*prefix*.hap1.p\_ctg.gfa). This graph keeps the phased
   paternal/haplotype1 assembly.

3. Phased maternal/haplotype2 contig graph (*prefix*.hap2.p\_ctg.gfa). This graph keeps the phased
   maternal/haplotype2 assembly.



In addition, hifiasm also outputs three binary files that save all overlap information (*prefix*.ec.bin, *prefix*.ovlp.reverse.bin, *prefix*.ovlp.source.bin). With these files, hifiasm can avoid the time-consuming all-to-all overlap calculation step, and do the assembly
directly and quickly. This might be helpful when you want to get an optimized
assembly by multiple rounds of experiments with different parameters.

Hifiasm is a standalone and lightweight assembler, which does not need external
libraries (except zlib). For large genomes, it can generate high-quality
assembly in a few hours. Hifiasm has been tested on various large and complex datasets. 
The results are as follows: 

|<sub>Dataset<sub>|<sub>GSize<sub>|<sub>Cov<sub>|<sub>Asm options<sub>|<sub>CPU time<sub>|<sub>Wall time<sub>|<sub>RAM<sub>|<sub> N50<sub>|
|:---------------|-----:|-----:|:---------------------|-------:|--------:|----:|----------------:|
|<sub>[\[Mouse (C57/BL6J)\]](https://www.ncbi.nlm.nih.gov/sra/?term=SRR11606870)<sub>|<sub>2.7Gb<sub>|<sub>x25<sub>|<sub>-t48 -l0<sub>|<sub>172.9h<sub>|<sub>4.8h<sub>|<sub>76G<sub>|<sub>20.6Mb<sub>|
|<sub>[\[Redwood\]](https://downloads.pacbcloud.com/public/dataset/redwood2020/)<sub>|<sub>26.5Gb<sub>|<sub>x23<sub>|<sub>-k 40 -t 64 -r 2<sub>|<sub>7274h30m<sub>|<sub>141h30m<sub>|<sub>512G<sub>|<sub>1.7Mb/1.9Mb<sub>|


## Usage

For Hifi reads assembly, a typical command line looks like:

```sh
./hifiasm -o NA12878.asm -t 32 NA12878.fq.gz
```

where `NA12878.fq.gz` is the input reads and `-o` specifies the output files.
In this example, all output files can be found at `NA12878.asm.*`. `-t` specifies 
the number of CPU threads. Note that at first run, hifiasm will save all overlaps 
to disk, which can avoid the time-consuming all-to-all overlap calculation next time. 
For hifiasm, once the overlap information has been obtained during the previous run 
in advance, it is able to load all overlaps from disk and then directly do assembly. 
If you want to ignore the pre-computed overlap information, please specify `-i`.

Please note that some old Hifi reads may consist of short adapters. To improve
the assembly quality, adapters should be removed by `-z` as follow:

```sh
./hifiasm -o butterfly.asm -t 42 -z 20 butterfly.fq.gz
```

In this example, hifiasm will remove 20 bases from both ends of each read.

For trio assembly, first the trio indexes of paternal/maternal should be generated by 
`yak count` (see https://github.com/lh3/yak):

```sh
./yak count -k31 -b37 -t16 -o mat.yak mat.fq.gz
```
```sh
./yak count -k31 -b37 -t16 -o pat.yak pat.fq.gz
```

and then run hifiasm as follow:

```sh
./hifiasm -o NA12878.asm -t 32 -1 pat.yak -2 mat.yak NA12878_1.fq.gz NA12878_2.fq.gz
```

[unitig]: http://wgs-assembler.sourceforge.net/wiki/index.php/Celera_Assembler_Terminology
[gfa]: https://github.com/pmelsted/GFA-spec/blob/master/GFA-spec.md
[paf]: https://github.com/lh3/miniasm/blob/master/PAF.md

## Getting Help

For detailed description of options, please see `man ./hifiasm.1`.
The `-h` option of hifiasm also provides simple description of options. If you
have further questions, please raise an issue at the issue page.

## Limitations and future works

1. The running time and memory usage should be further reduced.

2. The N50 should be further improved. 
