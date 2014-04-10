proteny
=======

A tool to analyze synteny at the protein level.
We develop an algorithm to detect statistically significant clusters of exons between two proteomes.
The tool provides algorithms to quickly detect and visualize results in order to support conclusions from genomic data.

The method discovers clusters of hits from a bi-directional BLASTp of translated exon sequences in two organisms.
A dendrogram for hits is built based on genomic distances between hits, and cut based on significance of a cluster score based on a permutation test at each node in the tree.
The result is a set of large clusters describing high exonic conservation.

![An example of the figures generated by proteny](/readme/example_output.gif)

Algorithm
=========

We use BLASTp to produce a set of hits, which are used to build

![We use BLASTp to produce a set of hits, which are used to build](/readme/clustering_dendrogram_a.gif)

a dendrogram which is traversed to find

![a dendrogram which is traversed to find](/readme/clustering_dendrogram_b.gif)

significant clusters

![significant clusters.](/readme/clustering_dendrogram_c.gif)

Which are then visualized in a useful way

![Visualizations with Circos](/readme/visualization.gif)


Dependencies
=============

Please make sure that the following packages are installed correctly before use.
(Circos is not necessary to run the program, but it is used to produce the visualizations)

 * CIRCOS: http://circos.ca/
 * IBIDAS: https://github.com/mhulsman/ibidas
 * BLAST+: ftp://ftp.ncbi.nlm.nih.gov/blast/executables/blast+/LATEST/

Installation
=========

Make sure that each dependency (and their dependencies) is installed.
modify the "$CIRCOS" variable in "circos_run" to reflect the location of your circos installation.

"example.py" contains an example which details the way the tool can be used.
"circos_run" is a script which runs circos finding each configuration file, to produce the PNG and SVG files.

Running the example
=====================

You need to download some files from the JGI DOE in order to run this example.
The files are accessible for free, but require an account:

 * http://genome.jgi-psf.org/Schco2/download/Schco2_AssemblyScaffolds.fasta.gz
 * http://genome.jgi-psf.org/Schco2/download/Schco2_GeneCatalog_genes_20110923.gff.gz

 * http://genome.jgi-psf.org/Agabi_varbisH97_2/download/Abisporus_varbisporusH97.v2.maskedAssembly.gz
 * http://genome.jgi-psf.org/Agabi_varbisH97_2/download/Abisporus_var_bisporus.mitochondrion.scaffolds.fasta.gz
 * http://genome.jgi-psf.org/Agabi_varbisH97_2/download/Abisporus_varbisporusH97.v2.FilteredModels3.gff.gz

Then, you can run the example:

```shell
$> ipcluster start -n 8 # Start the ipython cluster manager
$> ipython
In [1]: execfile("example.py");
$> cp circos_run PROTENY_OUTPUT/basid
$> ./circos_run
```

Usage
=========

You need to format each data source correctly.
In "data.py" there are defined some data sources already, which are processed into an IBIDAS format.
They take as input only a FASTA file and a GTF file.
As long as they are formatted correctly it is quite straight forward to read them with `Read()`
Suppose you have defined two data sources `org1()` and `org2()`, you can make a python file which runs Proteny:

```python

from utils import cluster_null as null;

from visualization import circos_data as cdata;
from visualization import circos_chr  as cc;

import proteny as ps;
import data;

#######################################

  # A function that visualizes what we want,
  # in this case, all chromosomes for for org1 
  # are shown against those in org2, one by one.
def viz(PR, k, outdir):

  files = cdata.write_data(PR, k, '%s/data' % circdir);
  KS = PR.key_s(k);

  bchrs = ["%s_%s" % (PR.org_names[k['id_b']], str(i)) for i in PR.org_genomes[k['id_b']].Get(0)() ];
  for i in xrange(PR.org_genomes[k['id_a']].Shape()()):
    cc.circos_chr(files, bchrs + ["%s_%d" % (PR.org_names[k['id_a']], i+1)], ["%s_%d=0.4r" % (PR.org_names[k['id_a']], i+1)], circdir, "scaffold_%02d_%s" % (i+1, KS) );
    print "scaffold_%02d_%s" % (i+1, KS);
  #efor
#edef

#######################################

  # Get data
org1 = data.org1();
org2 = data.org2();

  # Prepare the proteny structure
PR = ps.proteny();

  # Add our organisms.
id_a = PR.add_org(*org1, isfile=False);
id_b = PR.add_org(*org2, isfile=False);


  # Run analysis
k = PR.analyze(id_a=id_a, id_b=id_b,                     # Perform analysis between the two organisms we added
               cut='deeper_greater',                     # Find the smallest p-value given a conservation ratio
               nd=null.cluster_null_score_strict_smart,  # Our null distribution
               alpha=0.05,                               # p-value threshold
               ngenes_threshold=2,                       # Dont consider a cluster if it doesn't contain enough genes (not synteny)
               conservation_ratio=1);                    # Conservation ratio requirement

  # Save! It's important!
PR.save(savename);

  # Produce circos visualizations
viz(PR, k, circdir);

```
