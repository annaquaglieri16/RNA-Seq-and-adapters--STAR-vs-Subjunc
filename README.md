
# Align RNA-Seq data with adapter contamination 

Below is a comparison of [STAR](https://github.com/alexdobin/STAR) and [Subjunc](http://bioinf.wehi.edu.au/subjunc/) in the way they deal with adapter contamination using their defaults parameters. 

## How the worries started

My classic RNA-Seq pipeline includes STAR as aligner. Before working with these data I had never paid too much attention at adapter contamination since the most recent split-aware aligners should take care of it. I simply noticed the issue since I was working with a set of samples that had been run on two different flowcells, using 100 bp reads on one flowcell and 125 bp reads on the other flowcell. When comparing the proportion of uniquely mapped reads on the 125 bp libraries vs the 100 bp libraries for every run of the sample I obtained the following worrying plot:

![mapped_initial_star](https://cloud.githubusercontent.com/assets/7087258/22636698/390b2ea4-ec91-11e6-85f9-1446f9f06b13.png)

For every sample there are two labels (every sample on every flowcell has two runs). This shows that the proportion of mapped reads is costantly lower for the 125 bp libraries than the 100 bp libraries.

## Where did my reads go? 

### Comparisons of some defaults parameters: STAR and Subjunc 

- **Soft-Clip Adapters**. Both STAR and Subjunc (Subread) soft clip adapter at the end of the reads. 
- **Singleton**. With Paired End (PE) reads libraries STAR by default only outputs properly paired reads while SubJunc also outputs only one of the two mates.
- **STAR fragment mapped filter**. By default STAR outputs a pair only if it can map at least the 66% of the initial fragment (e.g. 0.66 x 200 bp with 100 bp reads). You can tune this threshold by changing the default value for *****

### Read-through adapters

My adapter-related problems arose since I am working with libraris with a large number of short fragments, often also shorter than the read length. This is the reason why I observed a great adapter contamination in libraries with longer reads. Below is a vignette explaining this concepts:

![Read-ThroughAdapters](https://cloud.githubusercontent.com/assets/7087258/22636440/46eb5c62-ec8f-11e6-81b6-c8ee51b58c94.png)

This concept is also clearly explained at [QCfail.com](https://sequencing.qcfail.com/). 

Looking at the **fragments size distributions** across libraries is also really useful to spot problems. Below is the example of what I was oberseving after aligning the RNA-Seq libraries untrimmed with STAR (Isize collected with ![PicardTools](https://broadinstitute.github.io/picard/command-line-overview.html) and plot creates with ![MultiQC](http://multiqc.info/)). 

![isize](https://cloud.githubusercontent.com/assets/7087258/22636909/ca8cae4c-ec92-11e6-8551-eab42a35a67a.png)

A steep bump in correspondence of the read lengths (100 bp and 125 bp) attracted our attention!






