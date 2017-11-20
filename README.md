# svm-gsu

*A C++ framework for training/testing the Support Vector Machine with Gaussian Sample Uncertainty (SVM-GSU).*

This is the implementation code for the Support Vector Machine with Gaussian Sample Uncertainty (SVM-GSU), whose linear variant (LSVM-GSU) was first proposed in [1], and its kernel version, i.e., Kernel SVM with Isotropic Gaussian Sample Uncertainty (KSVM-iGSU), was first proposed in [2]. If you want to use one of the above classifiers, please consider citing the appropriate [papers](#references).

Below, there are given detailed guidelines on how to [build](#0-prerequisites-and-build-guidelines) the code,  [prepare](#1-files-format) the input data files to the appropriate format (example files are given accordingly), and [use](#2-usage) the built binaries for training and/or testing SVM-GSU.  A [toy example](#toy-example) is given as a +++ .... A [Visualization tool](#visualization-of-lsvm-gsuksvm-igsu) written in Matlab is also given, along with some illustrative 2D toy examples. Short presentations of [LSVM-GSU](#a-linear-svm-with-gaussian-sample-uncertainty-lsvm-gsu-1) and [KSVM-iGSU ](#b-kernel-svm-with-isotropic-gaussian-sample-uncertainty-ksvm-igsu-2) are given below. For more detailed discussion of the above classifiers, please refer to the corresponding [papers](#references).





## 0. Prerequisites and build guidelines

This framework is built in [C++11](https://en.wikipedia.org/wiki/C%2B%2B11) using the [Eigen](http://eigen.tuxfamily.org/index.php?title=Main_Page) library. The code was originally developed in GNU/Linux ([Arch Linux](https://www.archlinux.org/)) and has been tested on Arch Linux, Debian, and Debian-based (e.g., *Ubuntu) distributions. In order to build the code, you first need to install (or make sure that you have already installed in your system) the following: 

```
- gcc >= 4.8
- Eigen 3.3.4
```

##### Install gcc 

- Arch Linux: `sudo pacman -S gcc`
- Debian/Ubuntu: `sudo apt-get install gcc`

##### Install Eigen

- Arch Linux: `sudo pacman -S eigen`
- Debian/Ubuntu: `sudo apt-get install libeigen3-dev `


In order to build the code, after cloning (or downloading and unzipping) the repo, follow the steps

- `$ cd svm-gsu/`
- `$ cd build/`
- `$ make`

Granted that `gcc` and `Eigen` have been correctly installed in your system, the above build process should generate the binaries `gsvm-train` and `gsvm-predict` under the `build/` directory.





## 1. Files format

The training set of SVM-GSU consists of the following three parts:

- A set of vectors that correspond to the **mean vectors** of the input data (input Gaussian distributions),
- A set of matrices that correspond to the **covariance matrices** of the input data (input Gaussian distributions), and 
- A set of binary **ground truth** labels that correspond to input data class labels.

We adopt a [libsvm](https://www.csie.ntu.edu.tw/~cjlin/libsvm/)-like file, plain-text format for the input data files. More specifically, for the above data files, we follow the formats described below. 

### Mean vectors file format

Each line of the mean vectors file consists of an n-dimensional feature vector, identified by a unique document id (`doc_id`), given in the following format: 

```
<doc_id> 1:<value> 2:<value> ... j:<value> ... n:<value>\n
```

The framework supports sparse representation for the feature vectors, i.e., a zero-valued feature can be omitted. For instance, the following line

~~~
feat_i 1:0.1 4:0.25 16:0.6
~~~

corresponds to a 16-dimensional feature vector that is associated with the document id *feat_i*. An example mean vectors file can be found [here]().

### Ground truth file format

Each line of the ground truth file consists of a binary `label` (in {+1,-1}) associated with a document id (`doc_id`):

~~~
<doc_id> <label>\n
~~~

An example ground truth file can be found [here]().

### Covariance matrices file format

Each line of the covariance matrices file consists of a matrix, which is identified by a unique document id (`doc_id`), and whose entries are given in the following format:

~~~
<doc_id> 1,1:<value> ... 1,j:<value> ... 1,n:<value> ... i,1:<value> ... i,j:<value> ... i,n:<value> ... n,1:<value> ... n,j:<value> ... n,n:<value>
~~~

Similarly to the mean vectors file, the framework adopts a sparse representation format for the covariance matrices. For instance, the following line

~~~
feat_i 1,1:0.125 2,2:0.5 3,3:2.25
~~~

corresponds to a 3x3 diagonal matrix that is associated with the document id *feat_i*. An example covariance matrices file can be found [here]().

**Note:** The document id (`doc_id`) that accompanies each line of the above data files, is used to uniquely identify each input datum (i.e., a triplet of a mean vector, a covariance matrix, and a truth label that describes an annotated multi-variate Gaussian distribution). In that sense, there is no need to put input mean vectors, covariance matrices, and truth labels in correspondance. Furthermore, the framework will find the intersection between the given `doc_id`'s (i.e., the given mean vectors, covariance matrices, and truth labels) and will construct the training set appropriately. As an example, if the given data files are as follows:

*Mean vectors file:*

~~~
doc_1 1:0.13 3:0.12
doc_3 1:-0.21 2:0.1 3:-0.43
doc_5 1:0.11 2:-0.21
~~~

*Ground truth labels file:*

~~~
doc_5 -1
doc_3 +1
doc_6 -1
~~~

*Covariance matrices file:*

~~~
doc_3: 1,1:0.25 2,2:0.01 3,3:0.1
doc_5: 1,1:0.25 2,2:0.01 3,3:0.1
~~~

then the framework will consider only the input data with ids `doc_3` and `doc_5`. Clearly, for a binary classification problem, the training set should include at least two training examples with different truth labels.

### Model files

After the training process of an SVM-GSU (linear or with the RBF kernel) is complete, a model file is created (with a filename defined as a command line argument -- see [usage](#gsvm-train)) so as to be subsequently used by `gsvm-predict` -- see [usage](#gsvm-predict). The file formats for the linear and the kernel variants of SVM-GSU are respectively described below:

#### Linear model file (example)

As an example, the following linear model file describes a trained LSVM-GSU that has been trained using diagonal covariance matrices, `T=1000` SGD iterations and `k=10` examples per each iteration, and a regularization parameter `lambda=100.` The parameters `sigmA` and `sigmB` refer to the Platt scaling method used for transforming the outputs of the classifier (i.e., the decision values) into a probability distribution over classes (i.e., so as to be interpreted as a-posteriori probabilities that the testing samples belong to the classes to which they have been classified -- see Sect. 3.1 of [1]). Finally, the optimal parameters (i.e., the parameters of the separating hyperplane, `w` and `b`), are given in the last two lines of the file:

~~~
SVM-GSU Model_File
kernel_type linear
lambda 100
sigmA -22.0186
sigmB -2.3374
cov_mat_type diagonal
w 0.0150138 0.0116311 0.00887193 0.00982599 0.0124118 0.0104942 0.0108264 0.0120473 0.00936632 0.0133811 0.0101319 0.0148072 0.00840837 0.0109078 0.0155783 0.0124124
b -0.00285747
~~~

#### Kernel model file (example)

*Not available yet.*





## 2. Usage

The framework consists of two basic components, one for training a SVM-GSU model [(gsvm-train)](#gsvm-train), and one for evaluating a trained SVM-GSU model on a given dataset [(gsvm-predict)](#gsvm-predict). Their basic usage is described below. In any case, information about their basic usage can also be obtained by running `gsvm-train` or `gsvm-predict` with no command line arguments.

### gsvm-train

Usage:

~~~
gsvm-train [options] <mean_vectors> <ground_truth> <covariance_matrices> <model_file>
~~~

Options:

~~~
-v <verbose_mode>: Verbose mode (default: 0)
-t <kernel_type>: Set type of kernel function (default 0)
	0 -- Linear kernel
    2 -- Radial Basis Function (RBF) kernel
-d <cov_mat>: Select covariance matrices type (default: 0)
	0 -- Full covariance matrices
	1 -- Diagonal covariance matrices
	3 -- Isotropic covariance matrices
-l <lambda>: Set the parameter lambda of SVM-GSU (default 1.0)
-g <gamma>: Set the parameter gamma (default 1.0/dim)
-T <iter>: Set number of SGD iterations
-k <k>: Set SGD sampling size
~~~

### gsvm-predict

Usage:

~~~
gsvm-predict [options] <mean_vectors> <model_file> <output_file>
~~~

Options:

~~~
-v <verbose_mode>: Verbose mode (default: 0)
-t <ground_truth>: Select ground truth file
-m <evaluation_metrics>: Evaluation metrics output file
~~~

### Toy example

In [toy_example/](https://github.com/chi0tzp/svm-gsu/tree/master/toy_example) you may find a minimal toy example scenario where you will train a LSVM-GSU model and evaluate it on a testing set. The data of this toy example are under [toy_example/data/](https://github.com/chi0tzp/svm-gsu/tree/master/toy_example/data). To run the toy example code, execute the BASH shell-script `run_toy_example.sh` (after making it executable by `chmod +xrun_toy_example.sh `).






## A. Linear SVM with Gaussian Sample Uncertainty (LSVM-GSU) [1]

### Motivation

In our method we consider that our training examples are multivariate Gaussian distributions with known means and covariance matrices, each example having a different covariance matrix expressing the uncertainty around its mean. This is illustrated in the figure below

<p align="center">
  <img src=".images/svmgsu_motivation.jpg" width="300" alt="SVM-GSU's motivation"/>
</p>

where the shaded regions are bounded by iso-density loci of the Gaussians, and the means of the Gaussians for examples of the positive and negative classes are located at "x" and "o" respectively. A classical linear SVM formulation (**LSVM**) would consider only the means of the Gaussians as training examples and, by optimizing the soft margin using the hinge loss and a regularization term, would arrive at the separating hyperplane depicted by the dashed line. In our formulation (**LSVM-GSU**), we optimize for the soft margin using the same regularization but the *expected* value of the hinge loss, where the expectation is taken under the given Gaussians. By doing so, we take into consideration the various uncertainties and arrive at a drastically different decision border, depicted by the solid line.  For a detailed presentation of LSVM-GSU, please refer to [1].





## B. Kernel SVM with Isotropic Gaussian Sample Uncertainty (KSVM-iGSU) [2]

*Not available yet.*



## Visualization of LSVM-GSU/KSVM-iGSU

A visualization tool build in Matlab is available under XXX/





## References

[1] Tzelepis, Christos, Vasileios Mezaris, and Ioannis Patras. "Linear Maximum Margin Classifier for Learning from Uncertain Data." *IEEE Transactions on pattern analysis and machine intelligence* XX.YY (2017): pppp-pppp.

[2] C. Tzelepis, V. Mezaris, I. Patras, "*Video Event Detection using Kernel Support Vector Machine with Isotropic Gaussian Sample Uncertainty (KSVM-iGSU)*", Proc. 22nd Int. Conf. on MultiMedia Modeling (MMM'16), Miami, FL, USA, Springer LNCS vol. 9516, pp. 3-15, Jan. 2016.