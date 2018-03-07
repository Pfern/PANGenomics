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
