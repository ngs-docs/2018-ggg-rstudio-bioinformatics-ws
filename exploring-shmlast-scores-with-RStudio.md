# Exploring shmlast score distributions in RStudio

(Often you need to run long-running jobs and then explore the output later.
For short workshops, it isn't feasible to run 20-30 minute jobs so we've
precomputed the output used below.)

You'll be introduced to the output of
[shmlast](http://joss.theoj.org/papers/3cde54de7dfbcada7c0fc04f569b36c7),
which is implements an algorithm for discovering potential orthologs
between an RNA-seq assembly and a protein database.

For the full lesson, see: [the ANGUS 2017 tutorial](https://angus.readthedocs.io/en/2017/running-blast-large-scale.html)

## Digression: What is shmlast doing?

shmlast is going to compute putative orthologs between mouse
transcripts and cow proteins.  Orthologs are genes that duplicated
from speciation (i.e. are the "same" gene in cow and mouse) and
are presumed to have the same function, although that is a computational
inference that needs to be treated with care.

The computation of orthologs (or homologs more generally) is a core
step in annotating genomes and transcriptomes.  The reason is that you
don't automatically get gene assignments when you build a new genome
or transcriptome - you just get unidentified DNA or RNA sequence! And
then you have to name each transcript or gene, and generally people
want that name to be the same across species.  And that involves
computing orthologs.

One of the most common ways to compute ortholog assignments is to use
reciprocal-best-hit BLAST, in which you use BLAST to find the two
sequences that match each other best in the database.  However,
reciprocal best hit has a few problems in the face of complicated
evolutionary scenarios or deep RNA sequencing; from the supp. material
of [Aubry et al., 2014](http://journals.plos.org/plosgenetics/article?id=10.1371/journal.pgen.1004365),

```
[... reciprocal best hit ] perform[s] well when sequences are present
as single copy genes in the datasets being compared and perform[s] less
well when trying to distinguish highly similar sequence groups (such
as multi-copy genes) from each other.  The issue of multiple-copy
genes and near identical gene-groups is particularly relevant for the
analysis of transcriptome data. It is to be expected that following de
novo assemblies of RNAseq data, most gene loci will be represented by
multiple assembled transcript variants.
```

basically this is saying that in many realistic scenarios (most especially
multi-copy genes, but also multiple isoforms) reciprocal best hit is
too conservative and will ignore real orthologs.

So Aubry et al. invent *conditional* reciprocal best hit, which tries
to find close homolog *groupings* that can deal with multi-copy genes.
shmlast is a reimplementation of that, done by our Camille Scott in the
DIB Lab.

### Why does shmlast take "so long"?

shmlast takes about 15-30 minutes to run. Why so long??

Well, if you look at the ANGUS tutorial we're calculating all pairwise matches between 36,000 mouse transcripts
and 64,000 cow proteins!  So frankly it's amazing it works so fast in the
first place!

----

## Looking at the output

Like most bioinformatics software, shmlast produces *a lot* of output.
How do we explore the results?

The main output is `mouse.1.rna.fna.gz.x.cow.faa.crbl.csv`, which is a
Comma-Separated Value file that you can load into any spreadsheet
program.  If we look at the file by typing

```
gunzip -c data/mouse.1.rna.fna.gz.x.cow.faa.crbl.csv.gz > data/mouse.1.rna.fna.gz.x.cow.faa.crbl.csv
head data/mouse.1.rna.fna.gz.x.cow.faa.crbl.csv
```

at the command line we should see something like this:

```
E,EG2,E_scaled,ID,bitscore,q_aln_len,q_frame,q_len,q_name,q_start,q_strand,s_aln_len,s_len,s_name,s_start,s_strand,score
6.6e-24,9.8e-16,23.18045606445813,641897,109.65804469295703,89,1,390,"ref|NM_001013372.2| Mus musculus neural regeneration protein (Nrp), mRNA",64,+,89,389,ref|XP_005212262.1| PREDICTED: DNA oxidative demethylase ALKBH1 isoform X1 [Bos taurus],0,+,241.0
5.4e-194,4.4e-165,193.26760624017703,719314,605.7589445367834,313,0,331,"ref|NM_207235.1| Mus musculus olfactory receptor 358 (Olfr358), mRNA",0,+,313,313,ref|XP_607965.3| PREDICTED: olfactory receptor 1361 [Bos taurus],0,+,1365.0
2.8e-188,5e-160,187.5528419686578,423289,588.9868500580775,307,0,323,"ref|NM_146368.1| Mus musculus olfactory receptor 361 (Olfr361), mRNA",0,+,307,313,ref|XP_607965.3| PREDICTED: olfactory receptor 1361 [Bos taurus],0,+,1327.0
6.6e-183,5.6e-155,182.18045606445813,725159,572.2147555793716,307,0,318,"ref|NM_146622.1| Mus musculus olfactory receptor 360 (Olfr360), mRNA",0,+,307,313,ref|XP_607965.3| PREDICTED: olfactory receptor 1361 [Bos taurus],0,+,1289.0
5.4e-194,4.4e-165,193.26760624017703,719315,605.7589445367834,313,0,331,"ref|NM_207235.1| Mus musculus olfactory receptor 358 (Olfr358), mRNA",0,+,313,313,ref|XP_002691614.1| PREDICTED: olfactory receptor 1361 [Bos taurus],0,+,1365.0
2.8e-188,5e-160,187.5528419686578,423290,588.9868500580775,307,0,323,"ref|NM_146368.1| Mus musculus olfactory receptor 361 (Olfr361), mRNA",0,+,307,313,ref|XP_002691614.1| PREDICTED: olfactory receptor 1361 [Bos taurus],0,+,1327.0
6.6e-183,5.6e-155,182.18045606445813,725160,572.2147555793716,307,0,318,"ref|NM_146622.1| Mus musculus olfactory receptor 360 (Olfr360), mRNA",0,+,307,313,ref|XP_002691614.1| PREDICTED: olfactory receptor 1361 [Bos taurus],0,+,1289.0
4.8e-183,5.6e-155,182.3187587626244,373474,572.2147555793716,266,0,310,"ref|XR_001782298.1| PREDICTED: Mus musculus predicted gene 4786 (Gm4786), misc_RNA",29,+,266,266,ref|NP_001035610.1| 60S ribosomal protein L7a [Bos taurus],0,+,1289.0
3.2e-153,3.1e-138,152.4948500216801,643504,516.6020212552417,246,1,659,"ref|NR_003628.1| Mus musculus predicted gene 5766 (Gm5766), non-coding RNA",357,+,246,266,ref|NP_001035610.1| 60S ribosomal protein L7a [Bos taurus],0,+,1163.0
```
(note that you can scroll to the right within the text in this tutorial to see all the output.)

Here the columns are helpfully labeled, but it's still kind of a mess
to look at - we'll look at in more detail in R, instead of using the
command line.  The key bits are the `q_name`_ and `s_name` column, which
tell you which *query* and which *subject* sequences match each other.

How big is this file? Big!  You can calculate how many lines are
present by using the `wc` command, which will tell you how many lines,
words, and characters are in the file:

```
wc data/mouse.1.rna.fna.gz.x.cow.faa.crbl.csv
```

```
  132901  2918423 39291181 mouse.1.rna.fna.gz.x.cow.faa.crbl.csv
```

## Some points for discussion

* What evolutionary scenarios are there that complicate orthology and
  homology assignment?


If we want to take a look at the results of shmlast,
we can't page through that big text file and understand it usefully.
We have to use another program to look at it.  Here, we're going to use
R and RStudio!

## Enter some R commands

(Enter the below commands into RStudio, not the command line.)

Read the precomputed object below in to R, and name it something that
you might remember:

```
shmlast_out <- read.csv("data/mouse.1.rna.fna.gz.x.cow.faa.crbl.csv")
```

Now we can take a look at the data in a slightly nicer way!

As with
[visualizing BLAST scores](visualizing-blast-scores-with-RStudio.md),
this is now a dataframe.  Use the `head()` command to take a
look at the beginning of it all:

```
head(shmlast_out)
```

and View to get a nice view in RStudio:

```
View(shmlast_out)
```

Run dim as before:

```
dim(shmlast_out)
```

That's a big data frame! 132,900 rows (and 17 columns!)

Hist, as before:

```
hist(shmlast_out$E_scaled)
```

This is telling us that MOST of the values in the `E_scaled` column are
quite high.  What does this mean? How do we figure out what this is?

(Hint: the [shmlast documentation](https://github.com/camillescott/shmlast) should tell us! Go to that page and search for "To fit the model".)

So these are a lot of *low* e-values.  Is that good or bad?  Should we
be happy or concerned that 

We can take a look at some more stats -- let's look at the `bitscore` column:

```
hist(shmlast_out$bitscore) 
```

what are we looking for here? (And how would we know?)

(Hint: longer bitscores are better, but even bitscores of ~1000 mean a
nucleotide alignment of 1000 bp - which is pretty good, no? Here we really
want to rescale the x axis to look at the distribution of bitscores in the
100-300 range.)

We can also look at the length of the queries, which are the mouse sequences
in this case. 

```
hist(shmlast_out$q_len)
```
 Compare this to the bitscores... do things match? Are
most mouse sequences getting matched by something of equivalent length?

Well, we can ask this directly with `plot`:

```
plot(shmlast_out$q_len, shmlast_out$bitscore)
```

why does this plot look the way it does?  (This may take a minute to show
up, note!)

(The bitscores are limited by the length of the sequences! You can't get a
*longer* bitscore than you have bases to align.)

### Summary points

This is an example of initial *exploratory data analysis*, in which we poke
around with data to see roughly what it looks like.  This is opposed to
other approaches where we might be trying to do statistical analysis to
confirm a hypothesis.

Typically with small sample sizes (n < 5) it is hard to do confirmatory
data analysis or hypothesis testing, so a lot of NGS work is done for
*hypothesis generation* and then confirmed via additional experimental
work.

## Some questions for discussion/points to make:

* Why are we using R for this instead of the UNIX command line, or Excel?

  (We'll talk more about this later, too!)
  
  One important thing to note here is that we're looking at a pretty large
  data set - with ease.  It would be much slower to do this in Excel.
  
* What other things could we look at?

* Have we done a basic check of just *looking* at the data? Go back and
  look at the data frame!  Do the gene name assignments look right?
  
  How might we do this a bit more systematically, while still "looking"
  at things? Try googling 'choose random rows from data frame' and then
  run
  
  ```
  shmlast_sub = shmlast_out[sample(nrow(shmlast_out), 10),]
  View(shmlast_sub)
  ```
  
* What does the following code do?

  ```
  tmp <- subset(shmlast_out, q_len >= 8000 & q_len <= 11000 & bitscore <=2000)    
  functions <- tmp[, c("q_name", "s_name")]
  ```

  and do you notice anything interesting about the names?  (They're
  all predicted/inferred genes.) What does this suggest about that
  "quadrant" of the data?
