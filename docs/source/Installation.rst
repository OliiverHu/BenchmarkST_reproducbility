Installation
============

We strongly suggest to follow the installation instructions given by each method. We will also provide the .yaml file of our debugging environment in each subfolder.


ADEPT
=============

Software dependencies
---------------------


Installation
------------


Reference
------------
https://github.com/maiziezhoulab/ADEPT


STAGATE
=============

Software dependencies
---------------------
.. code-block:: python

   scanpy
   pytorch
   pyG
   
The use of the mclust algorithm requires the rpy2 package and the mclust package. See https://pypi.org/project/rpy2/ and https://cran.r-project.org/web/packages/mclust/index.html for detail.

The cell type-aware module has not been supported by STAGATE_pyG yet.

Installation
------------
Downloading STAGATE_pyG code from https://github.com/QIFEIDKN/STAGATE_pyG

.. code-block:: python

   cd STAGATE_pyG-main
   python setup.py build
   python setup.py install

.. code-block:: python

   import STAGATE_pyG

Reference
------------
https://github.com/QIFEIDKN/STAGATE_pyG