# Exercise 1: Learning vg on toy examples

### Learning objectives
In this exercise you learn how to

- find toy examples to work with,
- construct a graph using `vg construct`,
- visualize that graph using `vg view`,
- simulate NGS reads from a graph using `vg sim`,
- create an index structure needed for read mapping using `vg index`,
- map reads to the graph using `vg map`.

###Getting started
Make sure you have vg installed. It is already available on the course workstations. If you want to bulid it on your laptop, follow the instructions at the [vg homepage](https://github.com/vgteam/vg) (note that building vg and all submodules from source can take ~1h). In this exercise, you will use small toy examples from the `test` directory. So make sure you have checked out vg:

	git clone https://github.com/vgteam/vg.git

Now create a directory to work on for this tutorial:

	mkdir exercise1
	cd exercise1
	ln -s ../vg/test/tiny

### Constructing and viewing your first graphs
Like many other toolkits, vg is comes with many different subcommands. First we will use `vg construct` to build our first graph. Run it without parameters to get information on its usage:

	vg construct

Let's construct a graph from just one sequence in file `tiny/tiny.fa`, which looks like this:

	>x
	CAAATAAGGCTTGGAAATTTTCTGGAGTTCTATTATATTCCAACTCTCTG

To construct a graph, run

	vg construct -r tiny/tiny.fa -m 32 >tiny.ref.vg

This will create a (very boring) graph that just consists of a linear chain of nodes, each with 32 characters.

The switch `-m` tells vg to put at most 32 characters into each graph node. (You might want to run it with different values and observe the different results.) To visualize a graph, you can use `vg view`. Per default, `vg view` will output a graph in [GFA](https://github.com/GFA-spec/GFA-spec/blob/master/GFA1.md) format. By adding `-j` or `-d`, you can generate [JSON](https://www.json.org/) or [DOT](https://www.graphviz.org/doc/info/lang.html) output.

	vg view tiny.ref.vg
	vg view -j tiny.ref.vg
	vg view -d tiny.ref.vg

To work with the JSON output the tool [jq](https://stedolan.github.io/jq/) comes in handy. To get all sequences in the graph, for instance, try

	vg view -j tiny.ref.vg | jq .node.sequence
	ERIK: THIS GIVES ME AN ERROR:
	jq: error (at <stdin>:1): Cannot index array with string "sequence"

Next, we use graphviz to layout the graph representation in DOT format.

	vg view -d tiny.ref.vg | dot -Tpdf -o tiny.ref.pdf

View the PDF and compare it to the input sequence. Now vary the parameter passed to `-m` of `vg construct` and visualize the result.

Ok, let's build a new graph that has some variants built into it. First, take a look at at `tiny/tiny.vcf.gz`, which contains variants in (gzipped) [VCF](https://samtools.github.io/hts-specs/VCFv4.2.pdf) format.

	vg construct -r tiny/tiny.fa -v tiny/tiny.vcf.gz -m 32 >tiny.vg

Visualize the outcome.  

Ok, that's nice, but you might wonder which sequence of nodes actually corresponds to the sequence (`tiny.fa`) you started from? To keep track of that, vg adds a **path** to the graph. Let's add this path to the visualization.

	vg view -dp tiny.ref.vg | dot -Tpdf -o tiny.pdf

You find the output too crowded? Option `-S` removes the sequence labels and only plots node IDs.

	vg view -dpS tiny.ref.vg | dot -Tpdf -o tiny.pdf

Another tool that comes with the graphviz package is *Neato*. It creates force-directed layouts of a graph.

	vg view -dpS tiny.ref.vg | neato -Tpdf -o tiny.pdf

For these small graphs, the difference it not that big, but for more involved cases, these layouts can be much easier to read.

### Mapping reads to a graph
Ok, let's step up to a slightly bigger example.

	ln -s ../vg/test/1mb1kgp

This directory contains 1Mbp of 1000 Genomes data for chr20:1000000-2000000. As for the tiny example, we build one linear graph that only contains the reference sequence and one graph that additionally encodes the known sequence variation.

	vg construct -r 1mb1kgp/z.fa -m 32 -p >ref.vg
	vg construct -r 1mb1kgp/z.fa -v 1mb1kgp/z.vcf.gz -m 32 -p >z.vg

You might be tempted to visualize these graphs (and of course you are welcome to try), but they are sufficiently big already that neato can run out of memory and crash.  

In a nutshell, mapping reads to a graph is done in two stages: first, seed hits are identified and then a sequence-to-graph alignment is performed for each individual read. Seed finding hence allows vg to spot candidate regions in the graph to which a given read can map potentially map to. To this end, we need an index. In fact, vg needs two different representations of a graph for read mapping XG (a succinct representation of the graph) and GCSA (a k-mer based index). To create these representations, we use `vg index` as follows.

	vg index -x z.xg z.vg
	vg index -g z.gcsa -k 16 z.vg

Passing option `-k 16` tells vg to use a k-mer size of 16. The best choice of k will depend on your graph and will lead to different trade-offs of sensitivity and runtime during read mapping.  

As mentioned above, the whole graph is unwieldy to visualize. But thanks to the XG representation, we can now quickly **find* individual pieces of the graph. Let's extract the vicinity of the node with ID 2401 and create a PDF.

	vg find -n 2401 -x z.xg -c 10 | vg view -dp - | dot -Tpdf -o 2401c10.pdf

The option `-c 10` tells `vg find` to include a context of 10 nodes in either direction around node 2401. You are welcome to experiment with different parameter to `vg find` to pull out pieces of the graph you are interested in.  

Next, we want to play with mapping reads to the graph. Luckily, vg comes with subcommand to simulate reads off off the graph. That's done like this:

	vg sim -x z.xg -l 100 -n 1000 -e 0.01 -i 0.005 -a >z.sim

This generates 1000 (`-n`) reads of length (`-l`) with a substitution error rate of 1% (`-e`) and an indel error rate of 0.5% (`-i`). Adding `-a` instructs `vg sim` to output the true alignment paths in GAM format rather than just the plain sequences (**ERIK**: correct?). **ERIK**: For didactic reasons, it might be better to first work from the plain sequences and then output the GAM. Can sim created fastq? Or can map work on fasta input?  

We are now ready to map the simulated read to the graph.

	vg map -x z.xg -g z.gcsa -G z.sim >z.gam

**ERIK**: What's the best way to visualize the mapped reads? That might be good to add here.

For evaluation purposes, vg has the capability to compare the newly created read alignments to true paths of each reads used during simulation.

	vg map -x z.xg -g z.gcsa -G z.sim --compare -j

This outputs the comparison between mapped and and true locations in JSON format. **ERIK**: This needs some explanations. Can we add some command lines to illustrate how that is useful?

### Exploring the benefits of graphs for read mapping
To get a first impression of how a graph reference helps us do a better job while mapping reads. We will construct a series of graphs from a linear reference to a graph with a lot variation and look at mapping rates, i.e. at the fraction of reads that can successfully be mapped to the graph. For examples, we might include variation above given allele frequency (AF) cutoffs and vary this cutoff, which can be achieved as follows.

	vg construct -r 1mb1kgp/z.fa -v <(vcffilter -f 'AF > 0.5' 1mb1kgp/z.vcf.gz) -m 32 >z.AF0.5.vg
	vg construct -r 1mb1kgp/z.fa -v <(vcffilter -f 'AF > 0.1' 1mb1kgp/z.vcf.gz) -m 32 >z.AF0.1.vg
	vg construct -r 1mb1kgp/z.fa -v <(vcffilter -f 'AF > 0.01' 1mb1kgp/z.vcf.gz) -m 32 >z.AF0.01.vg
	vg construct -r 1mb1kgp/z.fa -v <(vcffilter -f 'AF > 0' 1mb1kgp/z.vcf.gz) -m 32 >z.AF0.vg

Alternatively, you can also use `bcftools` to subset the VCFs. The ``--exculde`` option in conjunction with custom [expressions](https://samtools.github.io/bcftools/bcftools-man.html#expressions) is particularly useful to this end.  

Next, we index all our new VCFs...

	vg index -x z.AF0.5.xg -g z.AF0.5.gcsa -k 16 -p z.AF0.5.vg
	vg index -x z.AF0.1.xg -g z.AF0.1.gcsa -k 16 -p z.AF0.1.vg
	vg index -x z.AF0.01.xg -g z.AF0.01.gcsa -k 16 -p z.AF0.01.vg
	vg index -x z.AF0.xg -g z.AF0.gcsa -k 16 -p z.AF0.vg
	#look at the sizes of the indexes and graphs
	ls -sh z.AF*

... and map reads to them while computing the avereage identity rate:

	vg map -d z.AF0.5 -G z.sim -j | jq .identity | sed s/null/0/ | awk '{i+=$1; n+=1} END {print i/n}'
	vg map -d z.AF0.1 -G z.sim -j | jq .identity | sed s/null/0/ | awk '{i+=$1; n+=1} END {print i/n}'
	vg map -d z.AF0.01 -G z.sim -j | jq .identity | sed s/null/0/ | awk '{i+=$1; n+=1} END {print i/n}'
	vg map -d z.AF0 -G z.sim -j | jq .identity | sed s/null/0/ | awk '{i+=$1; n+=1} END {print i/n}'

**ERIK**: So this is not about mapping read, as I wrote above, right? Maybe we can have participants rather dig into particular mappings and do this a bit more anecdotically. I still like a comparison to `bwa`.
