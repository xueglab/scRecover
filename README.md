# scRecover

*Zhun Miao*

*2018-05-30*

![logo](https://github.com/miaozhun/scRecover/blob/master/vignettes/scRecover_LOGO.png?raw=true)


# 1. Introduction

**`scRecover`** is an R package for **imputation of single-cell RNA-seq (scRNA-seq) data**. It will detect and impute dropout values in a scRNA-seq raw read counts matrix while **keeping the real zeros unchanged**.

**`scRecover`** employs the Zero-Inflated Negative Binomial (ZINB) model for dropout probability estimation of each gene and accumulation curves for prediction of dropout number in each cell. By combination with scImpute, SAVER and MAGIC, it not only detects dropout/real zeros **at higher accuracy**, but also **improve the downstream clustering and visualization results**.


# 2. Citation

If you use **`scRecover`** in published research, please cite:

> 


# 3. Installation

To install the *developmental version* from [**GitHub**](https://github.com/miaozhun/scRecover/):

```R
devtools::install_github("miaozhun/scRecover", build_vignettes = TRUE)
```

To load the installed **`scRecover`** in R:

```R
library(scRecover)
```


# 4. Input

**`scRecover`** takes two inputs: `counts` and one of `Kcluster` or `labels`.

The input `counts` is a scRNA-seq **read counts matrix** or a **`SingleCellExperiment`** object which contains the read counts matrix. The rows of the matrix are genes and columns are cells.

`Kcluster` is an integer specifying the number of cell subpopulations. This parameter can be determined based on prior knowledge or clustering of raw data. `Kcluster` is used to determine the candidate neighbors of each cell and need not to be very accurate.

`labels` is a character/integer vector specifying the cell type of each column in the raw count matrix. Only needed when `Kcluster = NULL`. Each cell type should have at least two cells for imputation.


# 5. Test data

Users can load the test data in **`scRecover`** by

```R
library(scRecover)
data(scRecoverTest)
```

The test data `counts` in `scRecoverTest` is a scRNA-seq read counts matrix which has 200 genes (rows) and 150 cells (columns).

```R
dim(counts)
counts[1:6, 1:6]
```

The object `labels` in `scRecoverTest` is a vector of integer specifying the cell types in the read counts matrix, corresponding to the columns of `counts`.

```R
length(labels)
table(labels)
```


# 6. Usage

## 6.1 With read counts matrix input

Here is an example to run **`scRecover`** with read counts matrix input:

```R
# Load test data for scRecover
library(scRecover)
data(scRecoverTest)

# Run scRecover with Kcluster specified
scRecover(counts = counts, Kcluster = 2, outputDir = "./outDir_scRecover/")

# Run scRecover with labels specified
scRecover(counts = counts, labels = labels, outputDir = "./outDir_scRecover/")
```

## 6.2 With SingleCellExperiment input

The [`SingleCellExperiment`](http://bioconductor.org/packages/SingleCellExperiment/) class is a widely used S4 class for storing single-cell genomics data. **`scRecover`** also could take the `SingleCellExperiment` data representation as input.

Here is an example to run **`scRecover`** with `SingleCellExperiment` input:

```R
# Load library and the test data for scRecover
library(scRecover)
library(SingleCellExperiment)
data(scRecoverTest)

# Convert the test data in scRecover to SingleCellExperiment data representation
sce <- SingleCellExperiment(assays = list(counts = as.matrix(counts)))

# Run scRecover with SingleCellExperiment input sce (Kcluster specified)
scRecover(counts = sce, Kcluster = 2, outputDir = "./outDir_scRecover/")

# Run scRecover with SingleCellExperiment input sce (labels specified)
scRecover(counts = sce, labels = labels, outputDir = "./outDir_scRecover/")
```


# 7. Output

Imputed expression matrices will be saved in the output directory specified by \code{outputDir} or a folder named with prefix 'outDir_scRecover_' under the current working directory when \code{outputDir} is unspecified.


# 8. Parallelization

**`scRecover`** integrates parallel computing function with [`BiocParallel`](http://bioconductor.org/packages/BiocParallel/) package. Users could just set `parallel = TRUE` (default) in function `scRecover` to enable parallelization and leave the `BPPARAM` parameter alone.

```R
# Load library
library(scRecover)

# Run scRecover with Kcluster specified
scRecover(counts = counts, Kcluster = 2, parallel = TRUE)

# Run scRecover with labels specified
scRecover(counts = counts, labels = labels, parallel = TRUE)
```

Advanced users could use a `BiocParallelParam` object from package `BiocParallel` to fill in the `BPPARAM` parameter to specify the parallel back-end to be used and its configuration parameters.

## 8.1 For Unix and Mac users

The best choice for Unix and Mac users is to use `MulticoreParam` to configure a multicore parallel back-end:

```R
# Load library
library(scRecover)
library(BiocParallel)

# Set the parameters and register the back-end to be used
param <- MulticoreParam(workers = 18, progressbar = TRUE)
register(param)

# Run scRecover with 18 cores (Kcluster specified)
scRecover(counts = counts, Kcluster = 2, parallel = TRUE, BPPARAM = param)

# Run scRecover with 18 cores (labels specified)
scRecover(counts = counts, labels = labels, parallel = TRUE, BPPARAM = param)
```

## 8.2 For Windows users

For Windows users, use `SnowParam` to configure a Snow back-end is a good choice:

```R
# Load library
library(scRecover)
library(BiocParallel)

# Set the parameters and register the back-end to be used
param <- SnowParam(workers = 8, type = "SOCK", progressbar = TRUE)
register(param)

# Run scRecover with 8 cores (Kcluster specified)
scRecover(counts = counts, Kcluster = 2, parallel = TRUE, BPPARAM = param)

# Run scRecover with 8 cores (labels specified)
scRecover(counts = counts, labels = labels, parallel = TRUE, BPPARAM = param)
```

See the [*Reference Manual*](https://bioconductor.org/packages/release/bioc/manuals/BiocParallel/man/BiocParallel.pdf) of [`BiocParallel`](http://bioconductor.org/packages/BiocParallel/) package for more details of the `BiocParallelParam` class.


# 9. Evaluation of scRecover

## 9.1 On downsampling data

We evaluated SAVER, scImpute, MAGIC and their combined with scRecover version SAVER+scRecover, scImpute+scRecover, MAGIC+scRecover on a downsampling scRNA-seq dataset generated by random sampling of reads from a SMART-seq2 scRNA-seq dataset (Petropoulos S, et al. Cell, 2016, https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-3929/).

### 9.1.1 Accuracy of dropout prediction

We found after combined with scRecover, scImpute+scRecover, SAVER+scRecover and MAGIC+scRecover will have higher accuracy than scImpute, SAVER and MAGIC respectively.

![](https://github.com/miaozhun/scRecover/blob/master/vignettes/Accuracy_legend.png?raw=true)
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/Accuracy.png?raw=true)

### 9.1.2 Predicted dropout number

We found scImpute+scRecover, SAVER+scRecover and MAGIC+scRecover will have predicted dropout numbers closer to the real dropout number than without combination with scRecover.

![](https://github.com/miaozhun/scRecover/blob/master/vignettes/Dropout_legend.png?raw=true)
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/Dropout.png?raw=true)

## 9.2 On 10X data

We applied the 6 imputation methods to a 10X scRNA-seq dataset (https://support.10xgenomics.com/single-cell-gene-expression/datasets/3.0.0/heart_1k_v3).

Then we measured the downstream clustering and visualization results by comparing to the cell labels originated from the dataset and deriving their Jaccard indexes.

We found a significant improvement of SAVER, scImpute and MAGIC after combined with scRecover.

<center>

#### Raw data
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_raw_data.png?raw=true)
<br><br>

#### SAVER
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_SAVER.png?raw=true)
<br><br>

#### SAVER + scRecover
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_SAVER+scRecover.png?raw=true)
<br><br>

#### scImpute
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_scImpute.png?raw=true)
<br><br>

#### scImpute + scRecover
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_scImpute+scRecover.png?raw=true)
<br><br>

#### MAGIC
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_MAGIC.png?raw=true)
<br><br>

#### MAGIC + scRecover
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/10x_t-SNE_MAGIC+scRecover.png?raw=true)

</center>

## 9.3 On SMART-seq data

Next, we applied the 6 imputation methods to a SMART-seq scRNA-seq dataset (Chu L, et al. Genome Biology, 2016, https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE75748).

Then we measured the downstream clustering and visualization results by comparing to the cell labels originated from the dataset and deriving their Jaccard indexes.

We found a significant improvement of SAVER, scImpute and MAGIC after combined with scRecover.

<center>

#### Raw data
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_raw_data.png?raw=true)
<br><br>

#### SAVER
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_SAVER.png?raw=true)
<br><br>

#### SAVER + scRecover
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_SAVER+scRecover.png?raw=true)
<br><br>

#### scImpute
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_scImpute.png?raw=true)
<br><br>

#### scImpute + scRecover
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_scImpute+scRecover.png?raw=true)
<br><br>

#### MAGIC
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_MAGIC.png?raw=true)
<br><br>

#### MAGIC + scRecover
![](https://github.com/miaozhun/scRecover/blob/master/vignettes/SMART-seq_t-SNE_MAGIC+scRecover.png?raw=true)

</center>


# 10. Help

Use `browseVignettes("scRecover")` to see the vignettes of **`scRecover`** in R after installation.

Use the following code in R to get access to the help documentation for **`scRecover`**:

```R
# Documentation for scRecover
?scRecover
```

```R
# Documentation for estDropoutNum
?estDropoutNum
```

```R
# Documentation for normalization
?normalization
```

```R
# Documentation for test data
?scRecoverTest
?counts
?labels
```

You are also welcome to contact the author by email for help.


# 11. Author

*Zhun Miao* <<miaoz13@mails.tsinghua.edu.cn>>

MOE Key Laboratory of Bioinformatics; Bioinformatics Division and Center for Synthetic & Systems Biology, TNLIST; Department of Automation, Tsinghua University, Beijing 100084, China.

