# a pangenome from viral quasispecies

A lab-based mix of RNA from 5 viral quasispecies. There are five quasispecies mixed together, but our reference has only 4 genomes in it.

Let's make a reference graph, align reads to it, and use surjection and k-means clustering to attempt to find the reads that map to the 5th genome. Finally, we'll assemble these and see if it matches known information about the fifth genome.

First we'll build and index the graph.

We build the graph using `vg msga` with the `-Z` flag to specify that the inputs sequences should be circularized in the graph. (Note that the assembly of these 4 genomes will always be circular, why is this?)

```
vg msga -f REF4.fasta -w 1024 -Z -D | vg mod -U 10 - | vg mod -c -  >REF4.vg
```

We then index the graph with xg:

```
vg index -x REF4.xg REF4.vg
```

And we prune and index with GCSA2:

```
vg index -g REF4.gcsa -k 16 -p <(vg mod -pl 16 -e 3 REF4.vg)
```

We then map the first 10k Illumina reads:

```
vg map -d REF4 -f <(zcat SRR961514_1.fastq.gz | head -40000) >SRR961514_1.first10k.gam
```

Our goal is to find the un-assembled genome within the Illumina reads. To do so we will use a technique based on obtaining the path-relative identity for each read against each of our 4 reference genomes. We hypothesize that the reads from the 5th genome will tend to have a unique pattern of relationships between the known genomes, and should show up in clustering analysis.

First we surject the reads into each one of the references and build a table of the output:

```bash
for ref in $(vg paths -L REF4.vg)
    do ( vg surject -x REF4.xg -p $ref SRR961514_1.first10k.gam | vg view -a - \
        | jq -cr '[.name, "'$ref'", if .identity == null then 0 else .identity end] | @tsv' ) | gzip >first10k.surj.$ref.tsv.gz
done
```

Now we join up the results to build one table:

```
( echo read.name id.896 id.HXB2 id.JRCSF id.NL43
join <(zcat first10k.surj.896.tsv.gz | cut -f 1,3 | sort -V) <(zcat first10k.surj.HXB2.tsv.gz | cut -f 1,3 | sort -V) \
| join  <(zcat first10k.surj.JRCSF.tsv.gz | cut -f 1,3| sort -V) - \
| join  <(zcat first10k.surj.NL43.tsv.gz | cut -f 1,3| sort -V) - ) \
| tr ' ' '\t' | gzip >first10k.surj.join.tsv.gz
```

In R we can run k-means clustering PCA to evaluate if there is a subset that doesn't match any of the references.
We take a number of steps to ensure that the PCA makes sense. First, we remove cases where we don't have a relatively good match for all of the references, as the number that don't are relatively few we consider these outliers.
Then, we normalize each row so that the identity sum is 1.

```R
require(tidyverse)
require(broom)
first10k <- (read.delim("first10k.surj.join.tsv.gz") %>% mutate(id.sum=id.896+id.HXB2+id.JRCSF+id.NL43) %>% subset(id.sum > 0.95*4))
first10k.pca <- prcomp(first10k[,2:5]/first10k$id.sum)
first10k.pca.df <- as.data.frame(first10k.pca$x)
first10k.pca.df$read.name <- first10k$read.name
first10k.kmeans <- kmeans(first10k[,2:5]/first10k$id.sum, 5)
first10k$cluster <- as.factor(first10k.kmeans$cluster)
first10k.pca.df$cluster <- as.factor(first10k.kmeans$cluster)
ggplot(first10k.pca.df, aes(x=PC1, y=PC2, color=cluster)) + geom_point()

```
