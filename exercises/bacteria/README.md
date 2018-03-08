# Taking bacterial pan-genomes to the sequence level

Traditionally, pan-genomes for bacteria are gene based. That is, you study the set of genes found at a certain taxonomic unit, see [Tettelin et al., 2015](http://dx.doi.org/10.1016/j.mib.2014.11.016) for a review.

In this practical, we explore ways of implementing such a gene-level pan-genome in the vg software ecosystem. This allows us to detect the presence/absence of genes as well as to model sequence variation within genes. This is important since key traits, such as drug resistance, can be conferred by both transfer of whole genes but also by specific mutations ([Blair et al., 2015](http://dx.doi.org/10.1038/nrmicro3380)).

## Data
Today, we will mostly work on E. coli data. On your workstations you find the following data sets:

- `data/bacteria/ncbi-genes` one file for each complete E. coli strain present at NCBI. These files contain one line per gene,
- `data/bacteria/reads` whole genome sequence data for a (random) subset of 10 E.coli strains for this study: (Earle et al., 2015)[http://dx.doi.org/10.1038/nmicrobiol.2016.41],
- `data/bacteria/contigs` the result of running the Minia3 assembler on the reads provided in the above directory.
- In case you need read data (and the corresponding assemblies) we can provide more (241 in total).

## Objectives
- Build a graph-based representation of a single gene (of your choice),
- Come up with ways of determining whether a gene is absent/present in a given sample,
- Build a whole bacterial pan-genome.
- Use this representation to find a set of essential genes (present in all strains) as opposed to accessory genes (missing in some strains).
