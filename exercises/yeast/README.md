# Yeast practical

using yeast data (NCYC) to work through alignment variant calling with changes in the reference graph

## reference data

From [Yue et al 2016](https://yjx1217.github.io/Yeast_PacBio_2016/data/) we get full genome assemblies of yeast:

```
wget http://hypervolu.me/~erik/tmp/yue2016_Assemblies.tar.gz
```

We also have a pangenome based on SNPs in the form of the SGRP2 project results.

```

```

## short read data

Illumina data:

```
sra.id      strain         species
SRR4074255  S288c          cerevisiae
SRR4074256  DBVPG6044      cerevisiae
SRR4074257  DBVPG6765      cerevisiae
SRR4074258  SK1            cerevisiae
SRR4074358  Y12            cerevisiae
SRR4074383  YPS128         cerevisiae
SRR4074384  UWOPS03-461.4  cerevisiae
SRR4074385  CBS432         paradoxus
SRR4074394  N44            paradoxus
SRR4074411  YPS138         paradoxus
SRR4074412  UFRJ50816      paradoxus
SRR4074413  UWOPS91-917.1  paradoxus
```

We can get the data for SK1 in FASTQ format from ENA:

```
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR407/008/SRR4074258/SRR4074258_1.fastq.gz
wget ftp://ftp.sra.ebi.ac.uk/vol1/fastq/SRR407/008/SRR4074258/SRR4074258_2.fastq.gz
```

Illumina data from NCYC strains that aren't in the SGRP2 or Yue2016 sets, from [NCYC open data](http://opendata.ifr.ac.uk/NCYC/). Let's get some of it from NCYC78 (a Saccharomyces cerevisiae of unknown origin collected in 1925), NCYC77 (a baker's yeast collected in 1921), and NCYC1187 (an ale production strain collected in 1990).

```
wget http://opendata.ifr.ac.uk/NCYC/NCYC78/NCYC78_Run1.tar.gz
wget http://opendata.ifr.ac.uk/NCYC/NCYC77/NCYC77_Run1.tar.gz
wget http://opendata.ifr.ac.uk/NCYC/NCYC1187/NCYC1187_Run1.tar.gz
```

