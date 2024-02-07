.. BenchmarkST documentation master file, created by
   sphinx-quickstart on Thu Sep 16 19:43:51 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

BenchmarkST - reproducibility documentation
=============================================================================

.. toctree::
   :maxdepth: 1
   :caption: Contents:

   Installation
   Data availability
   ADEPT
   GraphST
   conST
   conGI
   SpatialPCA
   DR-SC
   STAGATE
   CCST
   SEDR
   SpaGCN
   BayesSpace
   PRECAST
   BASS
   DeepST
   .. SPIRAL
   .. STAligner
   .. Paste
   .. Paste2
   .. PRECAST_integration
   .. SPACEL
   .. BASS_integration
   .. DeepST_integration
   

.. image:: ../Images/benchmarkst_pipeline.png
   :width: 600

News
========
online currently under development

TODO list
========
docs for general installation guide (1/1)

docs for data availability (1/1)

docs for clustering methods (8/14)

docs for integration methods (0/8)

docs for figure reproduction (0/n)

Introduction
========
We benchmarked 14 clustering methods and 8 integration methods all with state-of-the-art performance. Evaluation occurred on 9 publicly available ST datasets of varying sizes, technologies, species, and complexity. Different experimental metrics and analyses, like adjusted rand index (ARI), uniform manifold approximation and projection (UMAP), node and layer matching ratio, spatial coherence score (SCS), and downstream biological analysis, are meticulously designed to quantitatively and qualitatvely assess method performance as well as data quality. GraphST, ADEPT, STAGATE, and BASS showed overall superior performance across different experiments. However, most clustering algorithms performed well on some datasets but had lower accuracy on others. Paste/Paste2 demonstrated distinctive performance in downstream analyses.

Acknowledgement
========
We deeply appreciate for all the help that we have received from the authors of these fantastic algorithms!

Citation
========
Yunfei Hu, etc. "Benchmarking clustering, alignment and integration methods for spatial transcriptomics", currently under review
