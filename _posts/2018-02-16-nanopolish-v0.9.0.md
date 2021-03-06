---
layout: post
title: Nanopolish v0.9.0
author: jared
draft: false
comments: true
---

We've just pushed a new release of [nanopolish](https://github.com/jts/nanopolish) to github with two improvements to reduce runtime and improve usability. As many users have noticed the `nanopolish index` step is extremely slow. This program traverses a directory of fast5 files to link each read with its associated signal data. This required opening up every fast5 file to find the identifier of the read. In the new v0.9.0 release, nanopolish can use the `sequencing_summary.txt` files produced by the albacore basecaller as a hint to avoid opening every fast5. This removes the huge I/O burden and makes indexing much faster. In a small test on a recent sequencing run with 500,000 reads the time required for `nanopolish index` dropped from 8 hours to 30 minutes.

The second improvement in v0.9.0 is supporting multithreaded input. In the previous version we had a lock around our fast5 parsing code, serializing I/O. This was a bottleneck for some workflows, like `nanopolish call-methylation`, as the time required to read the fast5 files is greater than the time taken to run our methylation model. In the new release we rewrote the fast5 parsing code to be thread-safe, allowing us to remove the locks and process fast5 files in parallel. In a small benchmark on human chromosome 20 we find that methylation calling with 8 threads is roughly 6 times faster than the old version. The consensus caller benefits from multithreaded I/O as well and I was able to polish the E. coli genome in about 5 hours on a single computer with 8 cores (I think this benchmark result might be exaggerated though due to OS file caching).
