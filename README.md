
# Align RNA-Seq data with adapter contamination 

Below is a comparison of [STAR](https://github.com/alexdobin/STAR) and [Subjunc](http://bioinf.wehi.edu.au/subjunc/) in the way they deal with adapter contamination using their defaults parameters. 

### Some default comparisons

- **Soft-Clip Adapters**. Both STAR and Subjunc (Subread) soft clip adapter at the end of the reads. 
- **Singleton**. With Paired End (PE) reads libraries STAR by default only outputs properly paired reads while SubJunc also outputs only one of the two mates.
- **STAR fragment mapped filter**. By default STAR outputs a pair only if it can map at least the 66% of the initial fragment (e.g. 0.66 x 200 bp with 100 bp reads). You can tune this threshold by changing the default value for *****

Given that, my adapter-related problems arose since I am working with a library where a large number of short fragments, often also shorter than the read length. This is the reason why I observed a great adapter contamination in my libraries, especially for libraries with longer reads. Below is a vignette explaining this concepts:

![Read-ThroughAdapters](https://cloud.githubusercontent.com/assets/7087258/22636440/46eb5c62-ec8f-11e6-81b6-c8ee51b58c94.png)

