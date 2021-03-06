.. image:: adobo_logo.png

Welcome to adobo's documentation!
#################################

What adobo is
*************
adobo is an analysis framework for single cell RNA sequencing data (scRNA-seq) and enables exploratory analysis through a set of Python modules. adobo can be used to create analysis pipelines, facilitating analysis and interpetation of scRNA-seq data. The goal of adobo is to consolidate single cell computational analysis methods in the Python 3.x programming language.

Contact
*******
adobo is developed by Oscar Franzén at Karolinska Institutet. Limited support is available over e-mail: p.oscar.franzen@gmail.com

Notes on this document
======================
Some basic knowledge of Python, `NumPy <https://en.wikipedia.org/wiki/NumPy>`_ and `pandas <https://en.wikipedia.org/wiki/Pandas_(software)>`_ and their commonly used data structures is helpful but not necessary. Many of adobo's functions have parameters with sensible defaults so that all parameters don't need to be specified upon calling a specific function. For clarity, the most important parameters are shown in the examples below.

In this documentation the adobo package is referred to as 'ad'.

Finding tutorials
=================
Jupyter notebooks can be found `here <https://github.com/oscar-franzen/adobo/tree/master/notebooks>`_ (not complete yet).

Installation
============
Prerequisites
^^^^^^^^^^^^^
* At least Python 3.6
* PIP (Python Package Manager) 19.3.1

To use adobo the first step is to install it. The recommended way to install adobo is to use the Python package manager ``pip``, streamlining installation of package dependencies. First, make sure a recent version of ``pip`` is installed, and when this is done the repository can be cloned and installed (instead of using git to clone, it is also possible to just navigate to the `repository <https://github.com/oscar-franzen/adobo/>`_ and press the download button):

First make sure you are using an updated ``pip`` version:

.. code-block:: bash

   python3 -m pip --version

The above command should return 19.3.1 or higher. If it does not, upgrade through this command:

.. code-block:: bash

   python3 -m pip install --upgrade pip --user

Method 1 - downloading sources from the repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The first installation method requires that ``git`` is installed on your system. We can install adobo with the following commands:

.. code-block:: bash

   git clone https://github.com/oscar-franzen/adobo.git
   
   cd adobo
   
   python3 -m pip install . --user
   
   # cleanup
   cd ..
   rm -rf adobo

After installing it, you can now delete the cloned repository.

Method 2 - using the Python package index (PyPI)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
This will download the and compile the package and its dependencies from PyPI.

.. code-block:: bash

   python3 -m pip install adobo --user

Post-installation
^^^^^^^^^^^^^^^^^
To test that everything works, fire up up your Python 3 interpreter and try importing the library. It should greet you with the current version and the URL to the documentation (this page):

.. code-block:: bash

   # not necessary to set this environment variable,
   # but setting it prevents unexpected multithreading
   export OPENBLAS_NUM_THREADS=1
   
   python3

>>> import adobo as ad
adobo version 0.2.57. Documentation: https://oscar-franzen.github.io/adobo/

Package organization
====================
adobo is organized into several modules containing related functions. All module and function names are lowercase to make them easier to remember.

.. list-table::
   :header-rows: 1
   
   * - Module name
     - Meaning
   * - ``IO``
     - reading and writing data (input/output)
   * - ``data``
     - the dataset container class
   * - ``preproc``
     - data preprocessing such as filtering out cells and genes
   * - ``normalize``
     - normalization of raw read counts
   * - ``hvg``
     - highly variable gene discovery
   * - ``dr``
     - linear and non-linear dimensional reduction techniques
   * - ``clustering``
     - functions related to cell clustering
   * - ``de``
     - differential expression between cell clusters
   * - ``plotting``
     - data visualization
   * - ``bio``
     - functions related to biology, for example cell cycle and cell type prediction
   * - ``traj``
     - trajectory analysis
   * - ``bulk``
     - deconvolution methods for integrating bulk RNA-seq
   * - ``_stats``
     - miscellaneous or general statistical functions that don't fit anywhere else

Internal modules
^^^^^^^^^^^^^^^^
These do not need to be accessed but are listed here for documentation purposes.

.. list-table::
   :header-rows: 1

   * - Module name
     - Function content
   * - ``_colors``
     - related to color generation
   * - ``_log``
     - internal utilities
   * - ``_constants``
     - internal constants

Getting started and pre-processing your data
============================================
Loading the package
^^^^^^^^^^^^^^^^^^^
The first step is to load the adobo package by importing it:

.. code-block:: python3

   import adobo as ad

.. note::
   Debug information in the form of traceback output is suppressed by default. However, this information is often useful when trying to solve program bugs. To enable full traceback set:
   
   ``ad.debug=1``

Loading your data from a text file
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
First we need to create an instance of the data container class (an adobo object, :py:class:`adobo.data.dataset`). This will be a new object containing the single cell data, meta data and analysis results. The input file should be a gene expression matrix (rows as genes and cells as columns) in plain text format. Fields can be separated by any character and it can be changed with the ``sep`` parameter. ``sep`` can be a single character or a regular expression (default is the regular expression ``\s``). The data matrix file can have a header or not (``header=True`` indicates a header is present, otherwise use ``header=False``). :py:func:`adobo.IO.load_from_file` calls `datatable.fread <https://datatable.readthedocs.io/en/latest/>`_ and any additional parameters are passed on into this method. The function :py:func:`adobo.IO.load_from_file` is used to load data from a raw read counts matrix and the returned object is an instance of :class:`adobo.data.dataset`.

We can load compressed data directly without having to uncompress it first; the compression format is detected automatically (``gzip``, ``bz2``, ``zip`` and ``xz`` are indirectly supported through pandas). The `matrix market <https://math.nist.gov/MatrixMarket/formats.html#MMformat>`_ format is supported; it is expected that three files (``matrix.mtx.gz``, ``barcodes.tsv.gz``, and ``genes.tsv.gz``) are packed in one ``tar.gz`` archive.

In the below example we set ``bundled=True``, which tells adobo to search its internal ``data`` directory for the file. For normal data, ``bundled`` should be ``False`` (default). Here we use data (`GEO95315 <https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE95315>`_) from the mouse dentate gyrus generated using 10X Chromium:

.. code-block:: python3

   # example 1
   exp = ad.IO.load_from_file('GSE95315.tab.gz',
                              desc='mouse brain data',
                              sparse=True,
                              verbose=True,
                              bundled=True, # used to load example data
                              header=True,
                              sep='\t')

   # example 2
   exp = ad.IO.load_from_file('GSE95315.tab.gz', bundled=True)

``desc`` can be used to specify an arbitrary string describing the data, but it can also be left empty. The raw read counts matrix is stored in the attribute ``count_data`` inside the dataset object (:py:attr:`adobo.data.dataset.count_data`). By default, raw read counts are stored in a sparse data frame; sparsity can slow down the data loading step, but leaves a smaller memory footprint. Sparsity can be turned off by setting ``sparse=False`` in ``load_from_file``. Currently, the normalized data are not stored in a sparse matrix, because not all numpy vector operations can be applied on sparse matrices.

Your loaded data are stored in the attribute ``exp.count_data``, and after loading it is good practise to examine that the data were loaded properly:

>>> exp
Filename (input): /home/sinigang/adobo/data/GSE95315.tab.gz
Description: mouse brain data
Raw count matrix: 14,545 genes and 5,454 cells (filtered: 14,545x5,454)
Commands executed:
Normalizations available:
norm_data structure:

"Commands executed:" lists the commands which have been executed on the loaded data (it is empty right now since nothing has been applied yet). "Normalizations available:" lists normalization schemes applied on the data. "norm_data" structure shows the nested structure of the ``norm_data`` dict.

.. important::

   When loading data, this data should not previously have been normalized; i.e. it should be raw read counts. Non-integer values will trigger an error.

.. note::

   All downstream operations and analyses are performed and stored as attributes in the adobo object, i.e. functions are applied on this object.

   Many adobo functions have a ``verbose`` parameter, which when ``True`` makes the function more verbose. Furthermore, many functions have a ``retx`` parameter, which can be set to ``True`` to return the generated data in addition to storing it in the data object.

Creating the data class object directly from a data frame
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
In many cases we already have the data in a data frame, in those cases we can just create the container object directly:

.. code-block:: python3
 
   # where 'df' is the data frame, columns are cells and rows are genes
   exp = ad.dataset(df)

Saving object
^^^^^^^^^^^^^
It is convenient not having to repeat analyses once they are finished. Saving an object can be done via the ``joblib`` package (`complete joblib docs <https://joblib.readthedocs.io/en/latest/>`_; `pickle <https://docs.python.org/3/library/pickle.html>`_ is another option):

.. code-block:: python3

   import joblib
   
   # save objext (compress=0 will turn off data compression)
   joblib.dump(exp, 'test.joblib', compress=3)
   
   # load object
   exp = joblib.load('test.joblib')

Instead of writing three lines of code and always remembering the name of the output file, we can specify ``output_file`` in :py:func:`adobo.IO.load_from_file` and then calling :py:func:`adobo.data.dataset.save()`:

.. code-block:: python3

   exp = ad.IO.load_from_file('GSE95315.tab.gz',
                              desc='mouse brain data',
                              sparse=True,
                              verbose=True,
                              bundled=True, # used to load example data
                              header=True,
                              sep='\t',
                              output_file='test.adobo')
   
   # do something
   ...
   
   # save
   exp.save()

Accessing meta data for cells and genes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Both cells and genes can bave meta data. Meta data can easily be loaded into the data object. Meta data are stored in the adobo object (an instance of :py:class:`adobo.data.dataset`). Two data structures (instances of :class:`pandas.DataFrame`) hold meta data for cells and genes, respectively:

>>> exp.meta_cells
       total_reads status  detected_genes
C1            2746     OK            1513
C2            3655     OK            1236
C3            1245     OK             816
C4            1374     OK             761
C5            3063     OK            1124
...            ...    ...             ...
C5450         2402     OK            1375
C5451         1779     OK             989
C5452         2022     OK            1150
C5453         1868     OK            1200
C5454         1361     OK             884
[5454 rows x 3 columns]

>>> exp.meta_genes
               expressed  expressed_perc status mitochondrial  ERCC
C0                                                                 
0610007P14Rik       1897       34.781812     OK          None  None
0610009B22Rik       1219       22.350568     OK          None  None
0610009L18Rik        701       12.852952     OK          None  None
0610009O20Rik        341        6.252292     OK          None  None
0610010F05Rik        458        8.397506     OK          None  None
...                  ...             ...    ...           ...   ...
mt-Nd1              5399       98.991566     OK          None  None
mt-Nd2              4616       84.635130     OK          None  None
mt-Nd4              5284       96.883022     OK          None  None
mt-Nd5              2206       40.447378     OK          None  None
mt-Nd6               228        4.180418     OK          None  None
[14545 rows x 5 columns]

Adding meta data
^^^^^^^^^^^^^^^^
Meta data such as experimental factors (e.g. tissues, time points, batches, etc) can easily be added to your adobo object either by modifying the meta data structures directly:

.. code-block:: python3

   # df is a pandas Series, with the same length as the number of rows in exp.meta_cells
   # and sorted so that the order of the cells are the same as in exp.meta_cells
   exp.meta_cells['time_point'] = df

or by calling the function :py:func:`adobo.data.dataset.add_meta_data`, which takes four parameters:

1. ``axis`` can be either 'cells' or 'genes' depending on whether your added data represent data for cells or genes.
2. the ``key`` is used as variable name, choose something that makes sense, such as "tissue" for different tissues.
3. ``data`` should be a ``list``, a numpy array or a Pandas Series, containing your data with the same length as your ``axis`` (although if ``data`` is of type `pandas.Series <https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.html>`_ the length does not need to match as long as ``index`` is set in the Series).
4. ``type_`` (note, this one has a trailing underscore) indicates if the data are categorical or continuous (defaults to categorical).

Getting detailed help
^^^^^^^^^^^^^^^^^^^^^
All functions in adobo have full documentation, which is accessible as `docstrings <https://www.python.org/dev/peps/pep-0257/>`_ on the Python interactive console as well as online:

.. code-block:: python3

   help(ad)

   help(ad.IO.load_from_file)

Starting to explore your data
=============================
Finally it's time to start exploring the loaded data. A usual first step is to examine the number of reads per cell. Cells with an unusual high or low number of reads may be artifacts and can be filtered out. We can generate a histogram of the total number of reads of cells with the command:

.. code-block:: python3

   # These how's are supported: violin, boxplot and barplot.
   ad.plotting.overall(exp, what='reads', how='histogram', bin_size=100)

Which will generate the plot:

.. image:: reads_per_cell_histo.png

A vertical line can be included at any arbitrary threshold, for example to draw a red line at x=12,000 total reads by including the ``cut_off`` parameter:

.. code-block:: python3

   ad.plotting.overall(exp, how='histogram', cut_off=12000)

.. image:: hist_depth_cutoff_line.png

It can also be useful to examine the number of expressed genes per cell in order to detect any outliers:

.. code-block:: python3

   ad.plotting.overall(exp, what='genes', how='histogram', bin_size=100)

.. image:: genes_exp_per_cell_hist.png

A third informative plot is to relate the number of detected genes with the total read depth into a scatter plot:

.. code-block:: python3

   ad.plotting.overall_scatter(exp)

.. image:: overall_scatter.png

Detecting ERCC spikes
^^^^^^^^^^^^^^^^^^^^^
ERCC are known amounts of synthetic constructs added to RNA-seq libraries for quality control and normalization purposes :cite:`Jiang2011`. Not all experiments use ERCC spikes, but many do. The ERCC "genes" are usually prefixed with `ERCC-` in the gene expression matrix. If ERCCs are present, then we need to let adobo know about them so that these spikes are not included in downstream analyses. :py:func:`adobo.preproc.find_ercc` is used to flag the ERCC spikes (stored in the ``ERCC`` column of :py:attr:`adobo.data.dataset.meta_genes`):

.. code-block:: python3

   ad.preproc.find_ercc(exp, ercc_pattern='^ERCC[_-]\\S+$')

Detecting mitochondrial genes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Mitochondrial gene expression signals can serve to indirectly tell us how healthy the captured cells are. Dying and low quality cells tend to exhibit unusually high signal from these genes. One convenient function identifies mitochondrial genes in your data and adds the percent of mitochondrial gene expression to the cellular meta data. Often mitochondrial genes in the human and mouse genomes have gene symbols starting with the prefix ``mt-``, but this might vary from species to species.

.. code-block:: python3

   ad.preproc.find_mitochondrial_genes(exp, mito_pattern='^mt-')

Sometimes a regular expression is not possible and we can instead supply a list of gene IDs or symbols representing mitochondrial genes:

.. code-block:: python3

   ad.preproc.find_mitochondrial_genes(exp, genes=['geneA','geneB','geneC'])

Data filtering
==============
Applying simple filters
^^^^^^^^^^^^^^^^^^^^^^^
Simple filters refers to applying a strict minimum cutoff on the number of expressed genes per cell and the total read depth per cell. Simple filters are usually effective in removing low quality cells and uninformative genes. If your data come from Drop-seq, 10X, etc, requiring at least 1000 uniquely mapped reads per cell is often sufficient:

.. code-block:: python3

   ad.preproc.simple_filter(exp, minreads=1000, minexpgenes=0.001)

.. important::

   If your protocol is applying full-length mRNA sequencing, e.g. SMART-seq2, then your ``minreads`` threshold should be higher, for example 50000.

.. note::

   :py:func:`adobo.preproc.simple_filter` also has a `maxreads` parameter, which can be used to remove cells with an upper read count limit (perhaps useful for limiting doublets). However, this parameter is not set by default.

It is also desirable to remove genes with an expression signal in very few cells; such genes may contribute more noise than information. The ``minexpgenes`` parameter can be used to control how genes are filtered out. If you wish to not remove any genes at all, simply set it to zero:

.. code-block:: python3

   ad.preproc.simple_filter(exp, minreads=1000, minexpgenes=0)

Setting ``minexpgenes`` to a fraction indicates that at least that fraction of cells must express any gene. If ``minexpgenes`` is an integer it refers to the absolute number of cells that at minimum must express the gene for the gene not to be filtered out.

To reset all simple filters to original:

.. code-block:: python3

   exp.reset_filters()

Dynamic detection of low quality cells
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
A more sophisticated approach to detection of low quality cells is to use the function :py:func:`adobo.preproc.find_low_quality_cells`, which uses `Mahalanobis distance`_ to identify bad cells from five quality metrics.

.. important::

   ``find_low_quality_cells`` requires that there are ERCC spikes in your data.

The parameter ``rRNA_genes`` should either be a string containing the full path to a file on disk contaiing genes that are rRNA genes (the file should have one gene per line). ``rRNA_genes`` can also be a :py:class:`pandas.Series` object with gene symbols.

.. code-block:: python3

   ad.preproc.find_low_quality_cells(exp, rRNA_genes=rRNA)

Like all adobo functions, ``find_low_quality_cells`` modifies the passed object. However, ``find_low_quality_cells`` also returns a list of cells that are classified as low quality; to prevent such behavior simply assign the return to a variable:

.. code-block:: python3

   low_q_cells = ad.preproc.find_low_quality_cells(exp, rRNA_genes=rRNA)

Imputation of dropouts
^^^^^^^^^^^^^^^^^^^^^^
"Dropouts" are artifacts caused by the low amounts of mRNA in single cells, causing expressed genes to become undetected in the expression data. Droplet-based protocols tend to have a greater number of dropouts. Several statistical procedures have been developed to impute missing expression values. adobo implements the method introduced by Li *et al* :cite:`Li2018`. The original paper describes the theory; briefly, model parameters are estimated using the Gamma distribution, then it uses Elastic net regularization to fit a linear model, which is used to predict expression values of genes with zero expression. adobo's imputation function is :py:func:`adobo.preproc.impute`:

.. code-block:: python3

    ad.preproc.impute(exp, filtered=True, res=0.5, nworkers='auto', verbose=True)

The parameter ``filtered`` is used to indicate if imputation should run on the quality-filtered data or on the complete raw read count matrix (default is to run on the filtered). Parallelization is achieved using Python's ``multiprocessing`` module. The parameter ``nworkers`` can be used to set the number of worker processes, which can be 'auto' to autodetect this (although due to Python's `Global Interpreter Lock <https://www.dabeaz.com/python/UnderstandingGIL.pdf>`_ the number of workers should not be higher than the number of physical cores). Imputed data are stored in :py:attr:`adobo.data.dataset.imp_count_data`. If you have a low number of cells (<1000), set the cluster resolution parameter to something low, for example ``res=0.1``.

.. note::

   Runtime varies depending on number of physical cores and size of the dataset. Typical runtime for a dataset consisting of ~1000 cells using 10 cores is 15 min.

Normalization
=============
Normalization removes technical and sometimes experimental biases and is always necessary prior to analysis. Because a universal normalization scheme for scRNA-seq data is not available nor recommended, adobo supports several different procedures. The function :py:func:`adobo.normalize.norm` can be used to perform the following normalization methods:

**standard**
   Performs a standard normalization by scaling with the total read depth per cell and then multiplying with a scaling factor.

**rpkm**
   Normalizes read counts as Reads per kilo base per million mapped reads (RPKM) :cite:`Conesa2016`. This method should be used if you want to adjust for gene length, such as in a full-length mRNA protocol. To use this procedure you must first prepare a file containing combined exon lengths for genes; the file should contain two columns, **without a header**, and columns separated by one space. The following columns must be present: (1) gene symbols and (2) the sum of exon lengths. The filename is set with the ``gene_lengths`` parameter, which can also take a vector.

**fqn**
   Performs full quantile normalization :cite:`Bolstad2003`. FQN was a popular normalization scheme for microarray data. It is not very common in single cell analysis despite having been shown to perform well :cite:`Cole2018`. The present implementation does not handle ties well.

**clr**
   Centered log ratio normalization. This normalization scheme was introduced in Seurat version 3.0 :cite:`Stuart2018`. It is a simple normalization scheme and is an alternative to ``standard``.

**vsn**
   Variance stabilizing normaliztion based on a negative binomial regression model with regularized parameters. Introduced by :cite:`Hafemeister2019` and represents a more sophisticated normalization approach. Appears to marginally improve resolution. Can be used if you have UMI counts.

All normalization schemes can be followed by log-transformation by setting ``log=True``, which is default behavior. The log function can be set with the ``log_func`` parameter (default is numpy's log2 ).

To perform a ``standard`` normalization followed by ``log``-transformation, run:

.. code-block:: python3

   ad.normalize.norm(exp, method='standard')
   ad.normalize.norm(exp, method='clr')

The normalized data are stored in the attribute :py:attr:`adobo.data.dataset.norm_data`, which is a dictionary of dictionaries. If we run multiple normalizations they are all stored in the ``norm_data`` and we can use the name ``name`` parameter in :py:func:`adobo.normalize.norm` to give it a name (default name is the method). We can always call ``is_normalized()`` to determine if a dataset has been normalized:

>>> exp.is_normalized()
True

.. note::

   If you have previously executed :py:func:`adobo.preproc.find_ercc`, ERCC spikes will be normalized too, and these can be found in :py:attr:`adobo.data.dataset.norm_ercc`.

If you want to normalize using imputed data, then set the ``use_imputed`` to ``True``:

.. code-block:: python3

   ad.normalize.norm(exp, use_imputed=True, method='standard')

Examining analysis history
==========================
Downstream analyses are performed on the data object. At any time it's possible to examine what functions have been applied on data object by calling :py:func:`adobo.data.dataset.assays`:

>>> exp.assays()
Number of mitochondrial genes found: 0
Number of ERCC spikes found: 92 
Normalization method: <not performed yet> 
Has HVG discovery been performed? No

When we run a function, it often generates results, which are stored in your data object. Specifically, results are stored in the ``norm_data`` dictionary. ``norm_data`` is where all analysis results are stored and it has a nested structure that can be displayed interactively by giving the name of the data object.

Detection of highly variable genes
==================================
Many algorithms used in scRNA-seq analysis perform better when used on a subset of measured genes :cite:`Yip2018`; the goal of the feature selection step is usually to extract a set of highly variable genes (HVG). adobo currently implements the following strategies for HVG discovery:

**seurat** 
    The function bins the genes according to average expression, then calculates dispersion for each bin as variance to mean ratio. Within each bin, Z-scores are calculated and returned. Z-scores are ranked and the top 1000 are selected. Input data should be normalized first. This strategy was introduced in Seruat :cite:`Stuart2018`, it is simple yet highly effective in identifying HVG.

**brennecke**
    Implements the method described in :cite:`Brennecke2013`. ``brennecke`` estimates and fits technical noise using ERCC spikes (technical genes) by fitting a generalized linear model with a gamma function and identity link and the parameterization w=a_1+u+a0. It then uses the chi2 distribution to test the null hypothesis that the squared coefficient of variation does not exceed a certain minimum. False discovery rate (FDR)<0.10 is considered significant.

**scran**
    scran fits a polynomial regression model to technical noise by modeling the variance versus mean gene expression relationship of ERCC spikes (the original method used local regression) :cite:`Lun2016`. It then decomposes the variance of the biological gene by subtracting the technical variance component and returning the biological variance component.

**chen2016**
    This method uses linear regression, subsampling, polynomial fitting and gaussian maximum likelihood estimates to derive a set of HVG :cite:`Chen2016`.

**mm**
    Selection of HVG by modeling dropout rates using modified Michaelis-Menten kinetics :cite:`Andrews2018`. This method calculates dropout rates and mean expression for every gene, then models these with the Michaelis-Menten equation (parameters are estimated with maximum likelihood optimization). The basis for using MM is because most dropouts are caused by failure of the enzyme reverse transcriptase, thus the dropout rate can be modelled with theory developed for enzyme reactions. This implementation works best for libraries sequenced to saturation (i.e. not Drop-seq).

Example:

.. code-block:: python3

   ad.hvg.find_hvg(exp, method='seurat', ngenes=1000)

Dimensional reduction
=====================
These are techniques to reduce the number of dimensions under consideration.

Principal Component analysis (PCA)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
`PCA <https://en.wikipedia.org/wiki/Principal_component_analysis>`_ decomposition :cite:`Abdi2010` of single cell data is for the most part necessary prior to clustering. The reason for this is because the graph construction benefits from a strong signal from each feature. PCA computation in adobo is performed by invoking :py:func:`adobo.dr.pca()`. Scaling of the data should always be performed before PCA, and this is done by default (although it can be turned off by setting ``scale=False``). Two approaches are available for PCA decomposition, and it should not matter much which one is used:

**irlb**
    Computed via truncated singular value decomposition by implicitly restarted Lanczos bidiagonalization :cite:`Baglama2005`. `irlb` may be better at handling very large single cell datasets and it is the default.

**svd**
    The standard approach to PCA. Computed via singular value decomposition (svd). (More likely to raise ``MemoryError``.)

Examples:

.. code-block:: python3

   # 75 components are returned by default, may need to be adjusted depending on your dataset
   ad.dr.pca(exp, method='irlb', ncomp=75)
   
   # or...

   ad.dr.pca(exp, method='svd', ncomp=75)

We can now examine the top contributing genes to each PCA component by producing a plot with :py:func:`adobo.plotting.pca_contributors`.

To plot the top 10 contributing genes to the first five components:

.. code-block:: python3

   ad.plotting.pca_contributors(exp, dim=[0,1,2,3,4], top=10)

.. image:: pca_cont.png

We can also write the output to a file instead of showing it on the screen:

.. code-block:: python3

   ad.plotting.pca_contributors(exp, dim=[0,1,2,3,4], top=10, filename='top_pca_genes.pdf')

The default number of components, 75, is often sufficient. However, a more deterministic approach is to generate an elbow plot using :py:func:`adobo.plotting.pca_elbow`:

.. code-block:: python3

   ad.plotting.pca_elbow(exp)

.. image:: pca_elbow.png

t-Distributed Stochastic Neighbor Embedding (t-SNE)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
t-SNE :cite:`vanDerMaaten2008` is a non-linear dimensional reduction technique that optimizes for local distance. It is the *de facto* dimensional reduction technique used to visualize scRNA-seq data :cite:`Kobak2019`. adobo uses the scikit-learn implementation :cite:`Pedregosa2011`. The most important parameter is ``perplexity`` (related to number of nearest neighbors) and it can greatly influence how your plot looks like. Suggested values for ``perplexity`` lies between 5 and 50, and it is recommened to try higher values for datasets with more cells. Additional parameters (such as early_exaggeration, learning_rate, n_iter, and n_iter_without_progress) do not usually need to be specified but will be passed into :py:class:`sklearn.manifold.TSNE`. By default adobo runs t-SNE on the PCA decomposition:

.. code-block:: python3

   ad.dr.tsne(exp, perplexity=30, run_on_PCA=True, verbose=True)

The recommended approach is to run t-SNE on PCA components, but it can sometimes be informative to run it on your entire normalized expression matrix (this will take significantly longer time):

.. note::

   t-SNE is non-deterministic. Different runs can give different results. To get a more reproducible t-SNE plot consider setting the ``seed`` parameter to any random integer, which will generate reproducible random numbers.

Uniform Manifold Approximation and Projection (UMAP)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
UMAP :cite:`McInnes2018` is also used frequently to visualize scRNA-seq data, and it may be better at preserving the global structure of the data:

.. code-block:: python3

   ad.dr.umap(exp)

UMAP has a number of tunable parameters, for example ``n_epochs`` (try values between 100 to 2000).

Clustering
==========
A crucial step in scRNA-seq analysis is to group cells into clusters. Complex datasets consisting of thousands of cells can be reduced to a small number of clusters, which tend to be easier to analyze and interpret.

.. important::

   Clustering is performed in PCA space, therefore PCA components must have been calculated first using :py:func:`adobo.dr.pca`.

In adobo, clustering can be performed with a single line of code:

.. code-block:: python3

   # run leiden
   ad.clustering.generate(exp, distance='euclidean', res=0.8, clust_alg='leiden')
   
   # run louvain
   ad.clustering.generate(exp, distance='euclidean', res=0.8, clust_alg='louvain')
   
   # run walktrap
   ad.clustering.generate(exp, distance='euclidean', clust_alg='walktrap')

The above command will run the necessary steps to cluster your single cell dataset; the cluster membership vector is stored in :py:attr:`adobo.data.dataset.clusters`, i.e. represented by an array with the same length as the number of cells after pre-processing. adobo's default clustering algorithm first builds a Shared Nearest Neighbor graph :cite:`Ertoz2003` and then finds communities in this graph using the Leiden algorithm :cite:`Traag2019` (default). It may become necessary to change value of the ``res`` (resolution) parameter to find the optimal clustering outcome.

By default :py:func:`adobo.clustering.generate` will return a ``dict`` containing cluster sizes (number of cells), use ``retx=False`` to disable this behavior.

Other community detection algorithms are also supported via the `igraph <https://igraph.org/redirect.html>`_ package:

* ``louvain`` :cite:`Blondel2008`
* ``walktrap`` :cite:`Pons2006`
* ``spinglass`` :cite:`Reichardt2006`
* ``multilevel`` :cite:`Blondel2008`
* ``infomap`` :cite:`Rosvall2008`
* ``label_prop`` (label propagation) :cite:`Raghavan2007`
* ``leading_eigenvector`` (Newman's leading eigenvector method) :cite:`Newman2006`

Various parameters of the these algorithms can be changed in :py:func:`adobo.clustering.generate`.

.. note::

   ``spinglass`` does not scale well on large datasets :cite:`Yang2016`.

Clustering visualization
========================
Visualization of cells in 2d space is performed with the function :py:func:`adobo.plotting.cell_viz`. Before running ``cell_viz``, the appropriate reduction functions must have been invoked (t-SNE or UMAP; see above). ``cell_viz`` can plot custom meta data, gene expression, and clustering outcomes. Three parameters specified as tuples control what to plot:

``clustering``
    Default is 'leiden'.
``metadata``
    Specify variable(s) listed as meta data in :py:attr:`adobo.data.dataset.meta_cells` (added via :py:func:`adobo.data.dataset.add_meta_data`).
``genes``
    Specify any gene symbol(s).

The above parameters can be mixed and will in such cases generate one subplot for every specified variable. The ``ncols`` parameter can be used to set the number of columns in the plot (default is 2).

Examples:

.. code-block:: python3

   # tsne
   ad.plotting.cell_viz(exp, reduction='tsne', clustering='leiden', metadata='detected_genes')
   
   # plots 'leiden' by default
   ad.plotting.cell_viz(exp, reduction='umap')

Plotting mitochondrial gene expression:

.. code-block:: python3

   ad.plotting.cell_viz(exp, reduction='tsne', meta_data='mito_perc', clustering=(), genes=())

.. image:: mito_perc_tsne.png
   :align: center
   
Cell cycle prediction
=====================
It is often useful to get an understanding of cell cycle states in a new data set, and it boils down to predicting one of three cell cycle states for every cell: G1, S and G2M. An extreme simplification of the cell cycle is shown below (figure from Wikipedia).

.. image:: cell_cycle_wikipedia.png
   :align: center
   :alt: Source: Wikipedia

adobo contains a machine learning classifier based on the sklearn implementation of `Stochastic Gradient Descent <https://en.wikipedia.org/wiki/Stochastic_gradient_descent>`_. The classifier is trained on mouse embryonic stem cells (n=288 cells) :cite:`Buettner2015`. The original data can be retrieved from `here <https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-2805/>`_. Cell cycle classification is performed by calling two functions:

.. code-block:: python3

   # trains the classifier
   clf, tr_features = ad.bio.cell_cycle_train()
   
   # performs the actual classification of cells in your data
   ad.bio.cell_cycle_predict(exp, clf, tr_features)

Classification is stored in the attribute :py:attr:`adobo.data.dataset.meta_cells` in a column `cell_cycle`. The current design does not support prediction scores, although this can easily be changed by changing the `loss parameter <https://scikit-learn.org/stable/modules/sgd.html>`_.

We can visualize the results:

.. code-block:: python3

   ad.plotting.cell_viz(exp, metadata='cell_cycle', clustering=(), genes=())

.. important::

   Prediction is only valid on mouse data since the classifier is trained on mouse data. Make sure your gene expression contains Ensembl gene identifiers in one of the following two formats: ENSEMBL; GENESYMBOL_ENSEMBL.

Cell type prediction
====================
Runs marker-based cell type prediction. The predictable unit is a cell cluster, meaning that cell clustering must have been performed before prediction is called. Cell type prediction works on mouse and human data. Gene symbols must be ued in your data (not Ensembl identifiers). If your data use Ensembl identifiers, the function :py:func:`adobo.preproc.symbol_switch` can be used to switch to gene symbols.

The ``min_cluster_size`` parameter specifies which clusters to ignore (i.e. clusters with fewer cells than this are ignored). Will run on all normalizations and clusterings as default (or specify which one with ``name`` and ``clustering``).

.. code-block:: python3

   ad.bio.cell_type_predict(exp, min_cluster_size=10, verbose=True)

The output is stored in the attribute :py:attr:`adobo.data.dataset.norm_data`.

Differential expression (DE)
============================
DE analysis between is usually a key goal in the analysis workflow. In adobo, DE analysis can be performed between clusters or experimental groups (as defined by meta data). Currently, DE can be performed using linear models or Mann-Whitney U test (also called Wilcoxon rank-sum test). Linear models are significantly faster than performing individual hypothesis tests, but underlying assumptions may be violated and cause inflation of p-values. It is therefore sometimes preferred to use a non-parametric test, which albeit robust is substantially slower.

For linear models, significance is calculated from coefficients using t statistics.

To execute a LM-driven DE analysis:

.. code-block:: python3

   ad.de.linear_model(exp)

The above command runs pairwise comparisons between all clusters.

We might prefer to use the Mann–Whitney U test:

.. code-block:: python3

   # if nworkers is auto, the complete number of physical cores will be used
   # logical cores will likely not result in speedup due to Python's thread safety
   ad.de.wilcox(exp, nworkers='auto')

After running pairwise tests, a next typical goal is to join p-values so that each cluster receives a single list of marker genes. P-value merging can be achieved the following call:

.. code-block:: python3

   ad.de.combine_tests(exp, method='fisher', mtc='bonferroni', retx=True, verbose=True)

The above generate a set of marker genes for every cluster using `Fisher's method <https://en.wikipedia.org/wiki/Fisher%27s_method>`_.

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`

.. _scRNA-seq: https://en.wikipedia.org/wiki/Single_cell_sequencing
.. _Mahalanobis distance: https://en.wikipedia.org/wiki/Mahalanobis_distance

References
==========
.. bibliography:: references.bib
   :style: unsrt
