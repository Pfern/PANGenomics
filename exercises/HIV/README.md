# A Virus Pan-Genome

Today you will work in more self-directed fashion. The driving questions are:

- Can sequence graphs help us to analyze viral quasispecies data?
- How can we construct a virus pan-genome?
- Will the resulting read alignments have fewer artifacts?
- How can we analyze the diversity in a sample using pan-genomic approaches?

The data were are using for this is based an artifical lab mix of five viruses:

- https://github.com/cbg-ethz/5-virus-mix
- Di Giallonardo, et al., Nucleic Acids Research, Volume 42, Issue 14, pp. e115, 2014. [https://doi.org/10.1093/nar/gku537](https://doi.org/10.1093/nar/gku537)

The five reference sequences underlying the mix can be find in the above git repository, see at https://github.com/cbg-ethz/5-virus-mix/blob/master/data/REF.fasta.

At home, you can rely on sra-tools (more specifically `fastq-dump`) to download data from SRA. At GTPB, Illumina and PacBio data is already available at your workstations.

More concretely, we ask you to work on the following tasks in groups of four (in any order, based on your groups preferences):

- Study the effect of using a pan-genone reference on the read alignments. First, spot some examples of regions of high sequence diversity within the reads.  Construct a pan-genome representation of the five reference genomes, use it to align reads to it, surject them to a linear reference and compare the results to BWA. Second, think of ways to quantify the quality of the alignments across all reads and study the effect of different strategies for pan-genome construction on that measure.

- Analyze the genetic diversity in the five virus mix. Assume you don't know that five viruses went into that sample. Think of ways to visualize the structure of the data sets (Illumina and PacBio). Could you have inferred the number of present viruses from your visualization? Discuss why this is (or is not) possible.

### Presentations
We ask each group to

- Prepare a short (~5-10min) presentation on what they found.
- Create a log (i.e. in markdown) of the steps you took to get there to allow others to retrace your solutions.

### Tools that can be helpful today
There is a number of VG subcommands that can aid your tasks today, in particular:

- vg msga
- vg surject
- vg vectorize
- vg mod
- vg augment

### Hints

These are things we ran into during the practical, some of which may not have been obvious.

#### vg viewing

Rendering the HPV graphs using graphviz can be difficult. One way to reduce the difficulty of rendering (in terms of runtime and memory) is to change the rendering in vg view. Several options can help. First, you can use `vg view -d` alone, not rendering the paths as this can make the renderng much more complex. Secondly, you can add the "simple" mode flag, which removes the node labels, `vg view -dS` or `vg view -dpS`.

You may find that large PDFs cannot be read by different PDF viewers like evince. As a workaround, the

#### Surjection

In the case of the HIV graph, you have to specify the path that surject maps into. Otherwise, there is a bug that will cause a crash. In any case, this is a hard problem for surject to solve on its own as there are now many reference paths that overlap and it's not clear which it should work on.

To surject a paired-end alignment set, use `vg map -d ref -f mate1.fq.gz -f mate2.fq.gz | vg surject -x ref.xg -p REF_NAME -i -b - >aln.bam`.

#### Read depth

There are 1.5M Illumina reads in SRR961514. It's not necessary to use all of them in your analysis, as mapping this number of reads is no longer interactive. A simple hack is to take the first N reads using something like `zcat SRR961514_1.fastq.gz | head -$(echo "N*4" | bc) | gzip >firstN_1.fastq.gz` on both of the pair files. The same pattern can be used to take a subset of the pacbio reads.

#### Where is the Env gene?

To locate genes, you might want to look at the [NCBI HXB2 annotations](https://www.ncbi.nlm.nih.gov/nucleotide/K03455). Here is the gff3 record for the env gene:

```
K03455.1	Genbank	CDS	6225	8795	.	+	0	ID=cds4;Parent=rna0;Dbxref=NCBI_GP:AAB50262.1;Name=AAB50262.1;gbkey=CDS;product=AAB50262.1;protein_id=AAB50262.1
```

#### Circular HIV genome confusion

An earlier version of the [ideas.md](https://github.com/Pfern/PANGenomics/blob/master/exercises/HIV/ideas.md) walkthrough suggested that you should circularize the HIV genome. This is not the case, and the apparent circularity of the REF.fasta results from either homology between the start and end of some of the strains or from the generation of the references based on plasmid resequencing.

#### GFA output and Bandage

`vg view` produces [GFA format](https://github.com/GFA-spec/GFA-spec) which is a community standard interchange format for sequence graphs. You can open this in `Bandage` which is installed on your systems. Try `vg view g.vg >g.gfa` and then run `Bandage` and open your graph `.gfa` file. You'll need to render it and then you can navigate it. Bandage does not show paths but can easily show large scale structures in the graph.

#### Pruning

You may have noticed that you cannot directly index the HIV graph generated from `vg msga` using `vg index`. The reason is that the graph contains regions of high complexity. GCSA2 indexing will enumerate _all_ kmers up to 256bp within the graph, and in regions of high complexity this may mean many (>billions) of kmers.

To resolve this problem, we "prune" the graph with `vg mod -pl 16 -e 3` or `vg prune`. Try pruning the graph and rendering it with `vg view -d` to see what happens to the structures within it under certain pruning modes. We pass the pruned graph to `vg index -g`, indexing the kmers in it. Then when we map we give `vg map` the full graph (in xg format) and the GCSA based on the pruned graph. The MEM finding uses the pruned graph, but when we do the local alignment we can efficiently work with the full, un-pruned graph, returning alignments in its space.

#### ClustalO input

#### Assembly options
Currently, there are two assemblers installed on the workstations: 

- [bcalm2](https://github.com/GATB/bcalm), which just builds a compact De Bruijn graph, that is a graph composed of unitigs. Use you can use the `convertToGFA.py` script to convert the output to GFA.

- [minia3](https://github.com/GATB/minia) is based on BCALM, but performs additional graph modification steps to get out longer contigs.


#### Installing R packages

In [ideas.md](https://github.com/Pfern/PANGenomics/blob/master/exercises/HIV/ideas.md) you are given some R scripts to try. These require several packages that may not be installed on your system. To install them, in R run:

```R
install.packages("tidyverse")
install.packages("devtools")
require(tidyverse)
require(devtools)
install_github("vqv/ggbiplot")
```

#### Hacky workaround for [vg issue #1503](https://github.com/vgteam/vg/issues/1503)

If you try to use `vg pack` on long read alignments you may get strange SDSL errors complaining about ranks not being in some range. If so, there is a [sed hack you can use to fix the problem](https://github.com/vgteam/vg/issues/1503). A real fix will be in shortly.
