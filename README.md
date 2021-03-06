Overall information
================

About
-----

The overall goal of hybrid assembly is to combine short and long reads to improve contiguity and retrieve better quality genomes.

Short reads:
\* Cheap
\* Accurate
\* Small contig size
\* Gaps

Long reads:
\* Relative inexpensive
\* High error rate

Data description (EDIT: metadata from Raul)
-------------------------------------------

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

UPDATE: A new version of the guppy basecaller was released, I performed basecalling using Chris' setup outlined above with the new version, a detailed procedure can be found under Long\_reads folder.

![](README_files/figure-markdown_github/tweet-from-cbrown-1.png)

Analyses tracks
---------------

### Microbial Ecosystems services

Increased metabolic potential towards: Micropollutants
Nutrients
Heavy metals

### Microdiversity

Evolutionary advantages, horizontal gene transfer.

DESMAN is a potential tool to get variant information from MAGs. Several tutorials, check out one from the <a href="https://github.com/chrisquince/StrainMetaSim" target="_blank">developers</a>, and an adaptation by <a href="https://github.com/rprops/MetaG_analysis_workflow/wiki/21.-DESMAN" target="_blank">Ruben Prop</a> from [Denef's lab](http://www-personal.umich.edu/~vdenef/index.htm) at UMICH.

<https://github.com/rprops/MetaG_analysis_workflow/wiki/21.-DESMAN>

InStrain
