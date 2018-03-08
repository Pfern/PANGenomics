# Taking bacterial pan-genomes to the sequence level

Traditionally, pan-genomes for bacteria are gene based. That is, you study the set of genes found at a certain taxonomic unit, see [Tettelin et al., 2015](http://dx.doi.org/10.1016/j.mib.2014.11.016) for a review.

In this practical, we explore ways of implementing such a gene-level pan-genome in the vg software ecosystem. This allows us to detect the presence/absence of genes as well as to model sequence variation within genes. This is important since key traits, such as drug resistance, can be conferred by both transfer of whole genes but also by specific mutations ([Blair et al., 2015](http://dx.doi.org/10.1038/nrmicro3380)).

## Data
Today, we will mostly work on E. coli data. On your workstations you find the following data sets:

- `data/bacteria/ncbi-whole-genomes` one file for each complete E. coli strain present at NCBI, containing the **complete reference genome** for that strain,
- `data/bacteria/ncbi-genes` one file for each of these NCBI E. coli strain containing one line per **gene**,
- `data/bacteria/reads` whole genome sequence data for a (random) subset of 10 E.coli strains for this study: [Earle et al., 2015](http://dx.doi.org/10.1038/nmicrobiol.2016.41),
- `data/bacteria/contigs` the result of running the Minia3 assembler on the reads provided in the above directory.
- In case you need read data (and the corresponding assemblies) we can provide more (241 in total).

## Objectives
- Build a graph-based representation of a single gene (of your choice),
- Come up with ways of determining whether a gene is absent/present in a given sample,
- Build a whole bacterial pan-genome.
- Use this representation to find a set of essential genes (present in all strains) as opposed to accessory genes (missing in some strains).

## Ideas
- Do a pooled assembly of the 10 strains
- Pool the contigs after assembly, potentially iterate the assembly process, i.e. build a cDBG from contigs
- Use variant calls to linear reference genome and use `vg construct` to build a pan-genome
- Use vg vectorize to distinguish core /accessory genome after mapping contigs into the graph (i.e. constructed from all of the assemblies)
- Use an existing polished reference genome as a starting point (e.g. to align contigs to and then augment)
- Start from all the known gene sequences and use vg msga to build gene models. The expectation is that the different genes would end up in different components. But this process might be slow, DBG-based methods might be faster. Maybe start from a subset.
- Place contigs along a reference genome to avoid spurious contig-to-contig alignments
- Map reads to the pan-genome model
- Select one gene (gyrA) and align all sequences of this gene (from NCBI) from all strains progressively
- Use this gene model for genotyping
- Count gene identify in NCBI gene set and take the most frequent one, map reads to it (make sure to filter out false positive mappings)

## Hints


### GFA input to vg from minia and bcalm

You can read the minia3 assemblies into GFA and then vg using these commands. (The same is true for bcalm, and can be done on the unitig or contig sets from both of these assemblers.)

First we convert the graph to GFA:

```
convertToGFA.py SRR3050857_merged.fastq.contigs.fa SRR3050857_merged.fastq.contigs.gfa 51
```

Note that we used kmer size 51 for the assemblies, and you will need to change this parameter to `convertToGFA.py` if you use a different k.

Now we fix up the ID space to make vg happy (it can't handle node id == 0) and feed the result into vg:

```
cat SRR3050857_merged.fastq.contigs.gfa \
    | awk '$1=="L" { $2 +=1 ; $4+=1 } $1=="S" { $2+=1 } { print }' | tr ' ' '\t' \
    | vg view -Fv - >SRR3050857_merged.fastq.contigs.vg
```

It's now possible to view the graph in GFA format in Bandage to ensure that the conversion worked.

```
vg view SRR3050857_merged.fastq.contigs.vg >SRR3050857_merged.fastq.contigs.+1.gfa
Bandage &
```

### Filtering alignments

You may want to filter alignments to only keep those with a positive score (that are mapped). One easy way to do this is to use [jq](https://stedolan.github.io/jq/). If we want to keep alignments that are aligned we can do the following:

```
vg view -a aln.gam | jq -cr 'select(.score > 0)' | vg view -JaG - >aln.filt.gam
```

### Combining graphs together

`vg` has the `join` command, but this is probably _not_ what you want as it does the strange operation of linking all the subgraphs to a single head node.

Instead, it's possible to combine graphs by concatenating them via `cat`. However, they _must have separate id spaces_ or they will overwrite each other. To make a single joint id space, use `vg ids -j`, passing the files you want to join together on the command line:

```
vg construct -v tiny/tiny.vcf.gz -r tiny/tiny.fa >tiny.vg
cp tiny.vg 1.vg; cp tiny.vg 2.vg; cp tiny.vg 3.vg
# these are the same
vg stats -lz 1.vg
cat 1.vg 2.vg 3.vg | vg stats -lz -
# joining the id space allows us to merge the graphs safely
vg ids -j 1.vg 2.vg 3.vg
# now we get a different, bigger graph that contains three copies of input graph
cat 1.vg 2.vg 3.vg | vg stats -lz -
```
