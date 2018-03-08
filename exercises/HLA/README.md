## The MHC region

The MHC (major histocompatiblity complex) is a region on chromosome 6 in the human genome. It is enriched for  genes of the immune system, including the HLA (human leukocyte antigen) genes. Unlike antibodies, the HLA genes do not undergo somatic mutation or rearrangement. However, they do display antigen to antibodies and T-cell receptors. Thus, they are part of the adaptive immune system. Accordingly, these genes are under strong diversifying selection; any haplotype that becomes too common is susceptible to epidemic if a disease agent evolves the capacity to evade it. For this reason, the MHC region is one of the most wildly polymorphic regions of the human genome. Moreover, it is enriched for non-trivial structural variation as well as point variants.

![MHC](http://www.sciscogenetics.com/wp-content/uploads/2013/05/MHC.png)

Because the MHC region is so polymorphic, it is one of the regions of the human genome that is not well-served by linear references. Recognizing this limitation, starting with with GRCh38 the Genome Reference Consortium (GRC) began distributing a set of alternate scaffolds for the MHC (and some other regions) along with the reference sequence. However, many genomics tools do not support these alternate scaffolds well yet. 

## Constructing an MHC graph

Graph genome tools offer one possibility for how we could use the alternate scaffolds. The first step is to build a graph. You have a few options for how you might accomplish this. The most obvious way is probably an alignment method. You have already seen `vg msga`, which is one option. However, you may also want to use a different MSA tool. `vg` has a method to construct a graph directly from a multiple sequence alignment in either MAF or Clustal format. The following example shows how you could do this using Clustal Omega.

    # use Clustal Omega to make a multiple sequence alignment
    clustalo -i sequences.fasta --outfmt clustal > mult_seq_aln.clustal
    
    # convert the multiple sequence alignment to a VG file
    # -M = input file, -F = format, -m = max node size
    vg construct -M mult_seq_aln.clustal -F clustal -m 32 > msa.vg

You are free to try this method, but be warned that the MHC region is large and it may overwhelm MSA tools that were designed for smaller sequences. We have provided you with the GRC's own MSA of the MHC alternate scaffolds in MAF format, but you should not feel bound to use it.

It should be noted that there are other methods that you could use to build a graph of the MHC region. For instance, you could find a VCF and use `vg construct -v`. You could also use an assembly tool to build an assembly graph and construct a VG from the GFA.

## Choosing MHC scaffolds

An interesting question to ask for a new sample is which MHC scaffold (or pair of scaffolds) will represent its MHC region best. In reality, any individual's MHC haplotypes are unlikely to match any of the MHC scaffolds well. However, they will have recombinant subsequences in common with the scaffolds. Moreover, previous studies have demonstrated benefits in downstream application for choosing more representative sequences for the reference, even if these sequences are imperfect (see Dilthey, et al. 2014). 

We have provided for you a FASTQ of the reads from the Platinum Genomes Project NA12878 sample that mapped to the MHC region. Of course, this dataset is already biased by mapping to a linear reference. If you would like to work with a more unbiased sample, we have also included a whole genome dataset. However, you will have to contend with the larger data volume and the fact that not all of these reads will map to the MHC graph. 

## Variant calling in VG

In order to choose an MHC scaffold, it might be useful to perform variant calling against the graph you construct. However, you should not that this is only one route forward. Think about how you would use the results before you devote too much time to it.

VG has two methods for genotyping variants, but in the interest of simplicity we will only present one here. Note that to output a VCF, you will need to have constructed your graph using a method that preserved the reference sequence as a path. 

Now we use the indexed reads to do the variant calling.

    vg genotype -G mapped.gam.idx -r pathname -v graph.vg 

If this is too slow for you, it is also possible to speed up the execution by building a RocksDB index of the GAM first with `vg index` and then passing that in as a positional argument in place of `-G`.

How could you use these variant calls to determine which MHC scaffold(s) are most appropriate for this sample?

