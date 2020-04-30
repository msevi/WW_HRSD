Hybrid Assembly
================

About
-----

The overall goal of hybrid assembly is to combine short and long reads to improve contiguity.

Short reads:

-   Cheap
-   Accurate
-   Small contig size
-   Gaps

Long reads:

-   Relative inexpensive
-   High error rate

Data description
----------------

Bi-monthly sampling of Virginia Initiative Plant of raw influent and treated effluent.

-   Raw: 100 mL
-   Treated: 1000 mL

Short read output:

-   Platform: Illumina NovaSeq

Long read output:

-   Platform: MinIon R9.5 flow cell
    Note: Basecalling to be performed offline.

Preprocessing strategy
----------------------

Short read:
1. Rename files and produce a linkage file
2. Use fastp with polyG and polyX trimming
3. Check for vector contamination with UniVec

Long read:
Basecalling with guppy was performed on google cloud by Chris and commands to set it up can be found <a href="https://gist.github.com/chrisLanderson/3f8443e5ab837c14c2249102343b6587" target="blank_">here</a>

The basecaller automatically trims all the adapter sequences and it removes sequences with a mean Q &lt; 7, plus maybe a few other things. Usually these reads are ready for most downstream processes.
