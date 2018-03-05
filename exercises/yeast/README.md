using yeast data (NCYC) to work through alignment variant calling with changes in the reference graph

## data

From [Yue et al 2016](https://yjx1217.github.io/Yeast_PacBio_2016/data/)

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

Illumina data from NCYC strains that aren't in the SGRP2 or Yue2016 sets, from [NCYC open data](http://opendata.ifr.ac.uk/NCYC/).

```
wget http://opendata.ifr.ac.uk/NCYC/NCYC78/NCYC78_Run1.tar.gz
```
