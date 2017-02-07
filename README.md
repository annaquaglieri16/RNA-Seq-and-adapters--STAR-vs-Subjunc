Tips on aligning RNA-Seq tips with adapter contamination: STAR vs Subjunc 
   
   * [How the worries started](#how-the-worries-started)
   * [Where did my reads go?](#where-did-my-reads-go)
      * [Comparisons of some defaults parameters: STAR and Subjunc](#comparisons-of-some-defaults-parameters-star-and-subjunc)
      * [Some notes on read-through adapters](#some-notes-on-read-through-adapters)
   * [Methods comparison](#methods-comparison)
      * [Results](#results)
   * [Tips learnt](#tips-learnt)

Below is a comparison of [STAR](https://github.com/alexdobin/STAR) and [Subjunc](http://bioinf.wehi.edu.au/subjunc/) in the way they deal with adapter contamination in RNA-Seq using their defaults parameters. 

# How the worries started

My classic RNA-Seq pipeline includes STAR as aligner. Before working with these data I had never paid too much attention at adapter contamination since the most recent split-aware aligners should take care of it. I simply noticed the issue since I was working with a set of samples that had been run on two different flowcells, using 100 bp reads on one flowcell and 125 bp reads on the other flowcell. When comparing the proportion of uniquely mapped reads with the 125 bp libraries vs the 100 bp libraries for every run of each sample I obtained the following worrying plot:

![mapped_initial_star](https://cloud.githubusercontent.com/assets/7087258/22636698/390b2ea4-ec91-11e6-85f9-1446f9f06b13.png)

For every sample there are two labels (every sample on every flowcell has two runs). This shows that **the proportion of mapped reads is costantly lower for the 125 bp libraries than the 100 bp libraries**.

# Where did my reads go? 

## Comparisons of some defaults parameters: STAR and Subjunc 

- **Soft-Clip Adapters**. Both STAR and Subjunc (or Subread) soft clip adapters at the end of the reads. 
- **Singleton**. With Paired End (PE) reads libraries STAR by default only outputs properly paired reads while SubJunc also outputs one of the two mates if only one can be mapped. However, STAR allows singleton when the length of one mate is less than 1/3 of the other mate (see this thread for more information [Singleton](https://groups.google.com/forum/#!searchin/rna-star/Singletons$20STAR$20controls$20the$20mapped$20length$2Fscore$20of$20the$20output$20reads$2C%7Csort:relevance/rna-star/K8yVdkTlWoY/-OWbbqx1AwAJ)).
- **STAR filters of mapped fragment length**. By default STAR outputs a pair only if it can map at least the 66% of the initial fragment (e.g. 0.66 x 200 bp with 100 bp reads). You can tune this threshold by changing the default value for --outFilterScoreMinOverLread and --outFilterMatchNminOverLread (see [STAR Google Group thread](https://groups.google.com/forum/#!topic/rna-star/qNlabqkKfx8)).

## Some notes on read-through adapters

My adapter-related problems arose since I was working with libraris with a large number of short fragments, often also shorter than the read length. This is the reason why I observed a great adapter contamination in libraries with longer reads. Below is a vignette explaining this concepts:

![Read-ThroughAdapters](https://cloud.githubusercontent.com/assets/7087258/22636440/46eb5c62-ec8f-11e6-81b6-c8ee51b58c94.png)

This concept is clearly explained at [QCfail.com](https://sequencing.qcfail.com/). 

Looking at the **fragments size distributions** across libraries is also really useful to spot several types of problems. Below is the example of what I was oberseving after aligning the untrimmed RNA-Seq raw reads with STAR (Isize collected with [PicardTools](https://broadinstitute.github.io/picard/command-line-overview.html) and plot created with [MultiQC](http://multiqc.info/)). 

![isize](https://cloud.githubusercontent.com/assets/7087258/22636909/ca8cae4c-ec92-11e6-8551-eab42a35a67a.png)

A steep bump (**loss of short fragments??**) in correspondence of the read lengths (100 bp and 125 bp) attracted our attention!

# Methods comparison

In order to fully understand what was going on, I took six samples (with every run on every flowcell) out of the initial set and I tried the following combinations:

- STAR untrimmed (as my initial run)
- STAR trimmed with [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) (min 8 bases match to remove adapter)
- STAR trimming the 25 bp of the 125 bp libraries (remove the bias by setting the same read length?)
- Subjunc untrimmed
- Subjunc trimmed with [Trimmomatic](http://www.usadellab.org/cms/?page=trimmomatic) (min 8 bases match to remove adapter)

## Results

![prop_mapped_star_runs](https://cloud.githubusercontent.com/assets/7087258/22636103/e2b17d50-ec8c-11e6-8943-806f1cca99d5.png)

![prop_mapped_SJ_runs](https://cloud.githubusercontent.com/assets/7087258/22637235/f0e6b14e-ec94-11e6-830e-1d427f197431.png)

![Nmapped_100bp](https://cloud.githubusercontent.com/assets/7087258/22637395/b5d932b0-ec95-11e6-9db9-69cf27b07fa1.png)

![Nmapped_125bp](https://cloud.githubusercontent.com/assets/7087258/22637397/b915722c-ec95-11e6-8be7-49d443760e75.png)


# Tips learnt

- STAR by default only outputs proper pairs following the logic that singletons in PE libraries have overall poorer quality. FASTQC reports of the STAR unmapped reads show that normally what happens is that the second mate in the pair is of poorer quality (this is only the most striking remark). Also, the proportion of unmapped reads is relatively small and negligible given the poor quality (~1-3% of the initial library size).

- STAR outputs a pair only if it can map at least the 66% of the initial fragment (e.g. 0.66 x 200 bp with 100 bp reads). However, STAR can be instructed to align singleton by tuning the --outFilterScoreMinOverLread and --outFilterMatchNminOverLread arguments < 0.50.

- Subjunc on the contrary aligns singleton by default, that is why it maps also the unmapped reads from STAR with a 70-80% mapping proportion.

- Trimming adapters improves the QC and increase the number of aligned reads in STAR (using default parameters) which is now similar to Subjunc. It is best to remove adapter contamination if aligning STAR with default parameters.

- Trimming the last 25bp of the reads improves QC and alignemnt but it does not as good as proper adapter trimming.



Thanks to [gh-md-toc](https://github.com/ekalinin/github-markdown-toc) for great easy way to create Table of Contents!





