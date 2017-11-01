**SNVDM: the Social Network Viewpoint Discovery Model**
======

## __Introduction__

This repository provides the source code and details to retrieve the data used in the paper *Users Are Known by the Company They Keep: Topic Models for Viewpoint Discovery in Social Networks* by Thibaut Thonet, Guillaume Cabanac, Mohand Boughanem, and Karen Pinel-Sauvagnat, published at CIKM '17. More details about this work can be found in the original paper, which preprint is available at https://www.irit.fr/publis/IRIS/2017_CIKM_TCBPS.pdf. 

The source code presented here is the Java implementation of a collapsed Gibbs sampler for the proposed model SNVDM/SNDVM-GPU and baselines TAM, SN-LDA, and VODUM -- see our paper for full reference to these latter models. The code is based on the JGibbLDA implementation of collapsed Gibbs sampling for LDA (http://jgibblda.sourceforge.net/).
This repository also contains details about the Twitter datasets used for the evaluation of models and baselines in our paper. These datasets were introduced in *Analyzing Discourse Communities with Distributional Semantic Models* by Igor Brigadir, Derek Greene, and Pádraig Cunningham, published at WebSci '15 (https://dl.acm.org/citation.cfm?id=2786470). The original collections are available at http://dx.doi.org/10.6084/m9.figshare.1430449. For any use of these datasets, please cite Brigadir et al's paper. As we performed a noise filtering step on the datasets to eliminate irrelevant tweets, we provide in this repository the IDs for the tweets and their retweets that were actually used in the evaluation -- tweet content is not directly provided due to Twitter Terms of Use strictly prohibiting redistribution thereof.

## __Content__

In this section, we detail the content of this repository.

* The directory **bin** contains the runnable jar files **snvdm-gpu.jar**, **tam.jar**, **sn-lda.jar**, and **vodum.jar** that can be executed to perform collapsed Gibbs sampling on models SNVDM/SNVDM-GPU, TAM, SN-LDA, and VODUM, respectively. The next sections detail how to use these jar files.
* The directory **data** contains 2 sub-directories, **indyref** (for the dataset on the 2014 Scottish Independence Referendum) and **midterms** (for the dataset on the 2014 US Midterm Elections), with each 3 files: **tweets.txt**, **retweets.txt**, and **user_labels.txt**. The first two files give the IDs of the tweets and their retweets, respectively. Note that some of these tweets may not be available anymore (and thus may not be retrieved) if their respective authors deleted them. The third file contains the mapping of users' Twitter ID to their groundtruth viewpoint label: yes/no for indyref and dem/rep (i.e., Democrat and Republican) for midterms. This list of users is the same as that provided by Bridagir et al. Note that the total number of users does not exactly match the numbers given in our paper (Table 2) because some users were left without any tweets after preprocessing and were thus discarded.
* The directory **lib** contains the libraries used in code. It contains the files **args4j-2.0.6.jar**, **commons-io-2.4.jar**, and **commons-math3-3.5.jar** which correspond to the Args4j library (http://args4j.kohsuke.org/), the Apache Commons IO library (http://commons.apache.org/proper/commons-io/), and the Apache Commons Math library (http://commons.apache.org/proper/commons-math/), respectively.
* The directory **src** contains the source code for the different models' collapsed Gibbs samplers, compressed in jar files: **snvdm-gpu-src.jar**, **tam-src.jar**, **sn-lda-src.jar**, and **vodum-src.jar**.
* The file **LICENCE.txt** describes the licence of our code.
* The file **README.md** is the current file.

## __How to run the code__

This section provides instructions on how to run collapsed Gibbs sampling for the different models, what it is taken as input and what is given as output.

### __Input file__

The input files used to run our collapsed Gibbs sampling programs are the same across the different models -- although some models may not use all available information, e.g., social interactions will not be used by TAM and VODUM. An input file contains on the first line the number of users in the dataset. Then each line contains the data for one user using the following format:
<pre><code>&lt;user_ID&gt;	&lt;user_groundtruth_label&gt;	&lt;word_11&gt; ... &lt;word_1N&gt;;interactedUponBy:&lt;sender_ID_11&gt; ... sender_ID_1M&gt;|&lt;word_21&gt; ... &lt;word_2N&gt;;interactedUponBy:&lt;sender_ID_21&gt; ... sender_ID_2M&gt;|...	recipient_ID_1 ... recipient_ID_P</code></pre>

The first field is the user ID (a string or a number). After a tabulation, the second field corresponds to the viewpoint label for this user -- used only in the evaluation performed at the end of the execution. Then, after another tabulation, follows the documents posted by the user. A document's words are separated by spaces. If a document was interacted upon by another user (i.e. a sender, following the terminology of our paper), the document's last word is followed by a semicolon (";"), any keyword (e.g., interactedUponBy), and a colon ":" after which the list of interacting users' IDs are written, separated by space. Documents posted by the user are separated by pipes ("|"). Finally, after another tabulation, appears the list of IDs (separated by spaces) for users on which the line's user interacted (i.e., the recipients).

Example:
<pre><code>hillary	democrat	i hate #gop;interactedUponBy:barack john|obamacare ftw;interactedUponBy:barack|i love thai food	bill barack</code></pre>

Note that in the case of VODUM the input file also needs to contain the part-of-speech category (0 or 1) for each token. Therefore, in the input file for VODUM (only), each word is followed by its part-of speech and both are separated by a colon (":"), e.g., <word_11>:<pos_category_11>.

### __SNVDM/SNVDM-GPU__

#### __Command line execution__

The collapsed Gibbs sampler for SNVDM or SNVDM-GPU is run using the following command:
<pre><code>$ java -jar bin/snvdm-gpu.jar [-beta &lt;double&gt;] [-gamma0 &lt;double&gt;] [-gamma1 &lt;double&gt;] [-mu &lt;double&gt;] [-delta0 &lt;double&gt;] [-delta1 &lt;double&gt;] [-alpha &lt;double&gt;] [-eta &lt;double&gt;] [-lambda &lt;double&gt;] [-tau &lt;int&gt;] [-ntopics &lt;int&gt;] [-nviews &lt;int&gt;] [-niters &lt;int&gt;] [-burnin &lt;int&gt;] [-lag &lt;int&gt;] [-hypsamp] [-nchains &lt;int&gt;] [-savestep &lt;int&gt;] [-topwords &lt;int&gt;] -dir &lt;string&gt; -dfile &lt;string&gt;</code></pre>

To use SNVDM instead of SNVDM-GPU, simply set tau = 0. Alternatively, one could set lambda = 0 but doing so leads to less efficient code so we do not recommend it.

The meaning of each parameter is detailed below:

* ``-beta <double>``: Value of &beta;, the concentration parameter for the symmetric Dirichlet prior on &phi;<sub>00</sub>, &phi;<sub>01</sub>, &phi;<sub>10</sub>, &phi;<sub>11</sub> (distributions over words). Default value: 0.01.
* ``-gamma0 <double>`` and ``-gamma1 <double>``: Values of &gamma;<sub>0</sub> and &gamma;<sub>1</sub>, the shape parameters for the Beta prior on &psi;<sub>0</sub> and &psi;<sub>1</sub> (distributions over routes). Default value: 1.0.
* ``-mu <double>``: Value of &mu;, the parameter for the symmetric Dirichlet prior on &xi; (distribution over interacting users). Note that the concentration parameter actually used is &mu;/U. Default value: 1.0.
* ``-delta0 <double>`` and ``-delta1 <double>``: Values of &delta;<sub>0</sub> and &delta;<sub>1</sub>, the shape parameters for the Beta prior on &sigma; (distribution over levels). Default value: 1.0.
* ``-alpha <double>``: Value of &alpha;, the parameter for the symmetric Dirichlet prior on &theta; (distribution over topics). Note that the concentration parameter actually used is &alpha;/T. Default value: 1.0.
* ``-eta <double>``: Value of &eta;, the parameter for the symmetric Dirichlet prior on &pi; (distribution over viewpoints). Note that the concentration parameter actually used is &eta;/V. Default value: 1.0.
* ``-lambda <double>``: Value (between 0 and 1) of the portion of ball (interaction) to add for related colors (users) in the Generalized Pólya Urn scheme. Default value: 0.5.
* ``-tau <int>``: Number of acquaintances to consider for each user (among those interacing the most with her) in the Generalized Pólya Urn scheme. Default value: 10.
* ``-ntopics <int>``: Number of topics (T). Default value: 10.
* ``-nviews <int>``: Number of viewpoints (V). Default value: 2.
* ``-niters <int>``: Number of iterations to perform for each chain. Default value: 1000.
* ``-burnin <int>``: Number of iterations before starting collecting samples. Default value: 500.
* ``-lag <int>``: Number of iterations between samples to collect. Default value: 50.
* ``-hypsamp``: Indicates that hyperparameters should be updated (using the auxiliary variable sampling technique) instead of being kept constant. Default value: true.
* ``-nchains <int>``: Number of chains (independent executions of the program, each with a different random initialization) to perform. Default value: 1.
* ``-savestep <int>``: Number of iterations (after burnin) between samples to save in the output files. If the savestep is greater than (niters - burnin), then only one sample (the sample for the last iteration) will be saved for each chain. Default value: 500.
* ``-topwords <int>``: Number of top words (most probable words for each word distribution) to output.
* ``-dir <string>``: Path to the directory containing the data file, and where the output files will be saved.
* ``-dfile <string>``: Name of the data file.

**Example:**
<pre><code>$ java -jar "bin/snvdm-gpu.jar" -beta 0.01 -gamma0 1 -gamma1 1 -mu 1 -delta0 1 -delta1 1 -alpha 1 -eta 1 -lambda 0.5 -tau 10 -ntopics 15 -nviews 2 -niters 1000 -burnin 500 -lag 50 -hypsamp -nchains 1 -savestep 500 -topwords 20 -dir "data/midterms" -dfile "midterms.dat"</code></pre>

#### __Output files__

The execution of the collapsed Gibbs sampler for SNVDM and SNVDM-GPU outputs the following files for each savestep (corresponding to a saved model):

* **&lt;model name&gt;.others**: This file contains the value of the hyperparameters (*beta*, *gamma0*, *gamma1*, *mu*, *delta0*, *delta1*, *alpha*, *eta*), other parameters (*lambda*, *tau*, *ntopics*, *nviews*). It also specifies the number of users in the collection (*nusers*), the size of the vocabulary (*nwords*), as well as the results of the evaluation in terms of various metrics (*perplexity*, *coherence*, *purity*, *inversePurity*, *nmi*, *bCubedPrecision*, *bCubedRecall*, *bCubedF*).
* **&lt;model name&gt;.avgphi00**, **&lt;model name&gt;.avgphi01**, **&lt;model name&gt;.avgphi10**, **&lt;model name&gt;.avgphi11**: These files contain the distributions over background words &phi;<sub>00</sub>, the distributions over viewpoint words &phi;<sub>01</sub>, the distributions over topic words &phi;<sub>10</sub>, and the distributions over viewpoint-topic words &phi;<sub>11</sub>, averaged over the different collected samples. 
* **&lt;model name&gt;.avgpsi0**, **&lt;model name&gt;.avgpsi1**: These files contain the general distribution over routes &psi;<sub>0</sub> and the topic-specific distributions over routes &psi;<sub>1</sub>, averaged over the different collected samples.
* **&lt;model name&gt;.avgxi**: This file contains the viewpoint-specific distributions over interacting users &xi;, averaged over the different collected samples.
* **&lt;model name&gt;.avgsigma**: This file contains the user-specific distributions over levels &sigma;, averaged over the different collected samples.
* **&lt;model name&gt;.avgtheta**: This file contains the user-specific distributions over topics &theta;, averaged over the different collected samples.
* **&lt;model name&gt;.avgpi**: This file contains the user-specific distributions over viewpoints &pi;, averaged over the different collected samples.
* **&lt;model name&gt;.words**: This file contains the most probable words in the distribution over background words &phi;<sub>00</sub>.
* **&lt;model name&gt;.vwords**: This file contains the most probable words in the distributions over viewpoint words &phi;<sub>01</sub>.
* **&lt;model name&gt;.twords**: This file contains the most probable words in the distributions over topic words &phi;<sub>10</sub>.
* **&lt;model name&gt;.vtwords**: This file contains the most probable words in the distributions over viewpoint-topic words &phi;<sub>11</sub>.
* **&lt;model name&gt;.classmap**: This file contains the mapping between the groundtruth label strings (from the input file) and their index used in the output files. The first line corresponds to the number of different viewpoint labels in the collection.
* **&lt;model name&gt;.usermap**: This file contains the mapping between the user ID strings (from the input file) and their index used in the output files. The first line corresponds to the number of different users in the collection.
* **&lt;model name&gt;.wordmap**: This file contains the mapping between the word strings (from the input file) and their index used in the output files. The first line corresponds to the number of different words in the vocabulary.

### __TAM__

#### __Command line execution__

The collapsed Gibbs sampler for TAM is run using the following command:
<pre><code>$ java -jar bin/tam.jar [-omega &lt;double&gt;] [-alpha &lt;double&gt;] [-beta &lt;double&gt;] [-delta0 &lt;double&gt;] [-delta1 &lt;double&gt;] [-gamma0 &lt;double&gt;] [-gamma1 &lt;double&gt;] [-naspects &lt;int&gt;] [-ntopics &lt;int&gt;] [-niters &lt;int&gt;] [-burnin &lt;int&gt;] [-lag &lt;int&gt;] [-hypsamp] [-nchains &lt;int&gt;] [-savestep &lt;int&gt;] [-topwords &lt;int&gt;] -dir &lt;string&gt; -dfile &lt;string&gt;</code></pre>

The meaning of each parameter is detailed below:

* ``-omega <double>``: Value of &omega;, the concentration parameter for the symmetric Dirichlet prior on &phi;<sub>00</sub>, &phi;<sub>01</sub>, &phi;<sub>10</sub>, &phi;<sub>11</sub> (distributions over words). Default value: 0.01.
* ``-alpha <double>``: Value of &alpha;, the parameter for the symmetric Dirichlet prior on &theta; (distribution over topics). Note that the concentration parameter actually used is &alpha;/T. Default value: 1.0.
* ``-beta <double>``: Value of &beta;, the parameter for the symmetric Dirichlet prior on &pi; (distribution over aspects). Note that the concentration parameter actually used is &beta;/A. Default value: 1.0.
* ``-delta0 <double>`` and ``-delta1 <double>``: Values of &delta;<sub>0</sub> and &delta;<sub>1</sub>, the shape parameters for the Beta prior on &sigma; (distribution over levels). Default value: 1.0.
* ``-gamma0 <double>`` and ``-gamma1 <double>``: Values of &gamma;<sub>0</sub> and &gamma;<sub>1</sub>, the shape parameters for the Beta prior on &psi;<sub>0</sub> and &psi;<sub>1</sub> (distributions over routes). Default value: 1.0.
* ``-naspects <int>``: Number of aspects (A). Default value: 2.
* ``-ntopics <int>``: Number of topics (T). Default value: 10.
* ``-niters <int>``: Number of iterations to perform for each chain. Default value: 1000.
* ``-burnin <int>``: Number of iterations before starting collecting samples. Default value: 500.
* ``-lag <int>``: Number of iterations between samples to collect. Default value: 50.
* ``-hypsamp``: Indicates that hyperparameters should be updated (using the auxiliary variable sampling technique) instead of being kept constant. Default value: true.
* ``-nchains <int>``: Number of chains (independent executions of the program, each with a different random initialization) to perform. Default value: 1.
* ``-savestep <int>``: Number of iterations (after burnin) between samples to be saved in the output files. If the savestep is greater than (niters - burnin), then only one sample (the sample for the last iteration) will be saved for each chain. Default value: 500.
* ``-topwords <int>``: Number of top words (most probable words for each word distribution) to output. Default value: 20.
* ``-dir <string>``: Path to the directory containing the data file, and where the output files will be saved.
* ``-dfile <string>``: Name of the data file.

**Example:**
<pre><code>$ java -jar "bin/tam.jar" -omega 0.01 -alpha 1 -beta 1 -delta0 1 -delta1 1 -gamma0 1 -gamma1 1 -naspects 2 -ntopics 15 -niters 1000 -burnin 500 -lag 50 -hypsamp -nchains 1 -savestep 500 -topwords 20 -dir "data/midterms" -dfile "midterms.dat"</code></pre>

#### __Output files__

The execution of the collapsed Gibbs sampler for TAM outputs the following files for each savestep (corresponding to a saved model):

* **&lt;model name&gt;.others**: This file contains the value of the hyperparameters (*omega*, *alpha*, *beta*, *delta0*, *delta1*, *gamma0*, *gamma1*), other parameters (*ntopics*, *naspects*). It also specifies the number of documents (in our setting this corresponds to the number of users) in the collection (*ndocs*), the size of the vocabulary (*nwords*), as well as the results of the evaluation in terms of various metrics (*perplexity*, *coherence*, *purity*, *inversePurity*, *nmi*, *bCubedPrecision*, *bCubedRecall*, *bCubedF*).
* **&lt;model name&gt;.avgphi00**, **&lt;model name&gt;.avgphi01**, **&lt;model name&gt;.avgphi10**, **&lt;model name&gt;.avgphi11**: These files contain the distributions over background words &phi;<sub>00</sub>, the distributions over viewpoint words &phi;<sub>01</sub>, the distributions over topic words &phi;<sub>10</sub>, and the distributions over viewpoint-topic words &phi;<sub>11</sub>, averaged over the different collected samples. 
* **&lt;model name&gt;.avgtheta**: This file contains the document-specific (here, user-specific) distributions over topics &theta;, averaged over the different collected samples.
* **&lt;model name&gt;.avgpi**: This file contains the document-specific (here, user-specific) distributions over viewpoints &pi;, averaged over the different collected samples.
* **&lt;model name&gt;.avgsigma**: This file contains the document-specific (here, user-specific) distributions over levels &sigma;, averaged over the different collected samples.
* **&lt;model name&gt;.avgpsi0**, **&lt;model name&gt;.avgpsi1**: These files contain the general distribution over routes &psi;<sub>0</sub> and the topic-specific distributions over routes &psi;<sub>1</sub>, averaged over the different collected samples.
* **&lt;model name&gt;.words**: This file contains the most probable words in the distribution over background words &phi;<sub>00</sub>.
* **&lt;model name&gt;.awords**: This file contains the most probable words in the distributions over aspect words &phi;<sub>01</sub>.
* **&lt;model name&gt;.twords**: This file contains the most probable words in the distributions over topic words &phi;<sub>10</sub>.
* **&lt;model name&gt;.atwords**: This file contains the most probable words in the distributions over aspect-topic words &phi;<sub>11</sub>.
* **&lt;model name&gt;.classmap**: This file contains the mapping between the groundtruth label strings (from the input file) and their index used in the output files. The first line corresponds to the number of different viewpoint labels in the collection.
* **&lt;model name&gt;.docmap**: This file contains the mapping between the document (here, user) ID strings (from the input file) and their index used in the output files. The first line corresponds to the number of different documents in the collection.
* **&lt;model name&gt;.wordmap**: This file contains the mapping between the word strings (from the input file) and their index used in the output files. The first line corresponds to the number of different words in the vocabulary.

### __SN-LDA__

#### __Command line execution__

The collapsed Gibbs sampler for SN-LDA is run using the following command:
<pre><code>$ java -jar bin/sn-lda.jar [-alpha &lt;double&gt;] [-beta &lt;double&gt;] [-delta &lt;double&gt;] [-gamma &lt;double&gt;] [-ntopics &lt;int&gt;] [-ncomms &lt;int&gt;] [-niters &lt;int&gt;] [-burnin &lt;int&gt;] [-lag &lt;int&gt;] [-hypsamp] [-nchains &lt;int&gt;] [-savestep &lt;int&gt;] [-topwords &lt;int&gt;] -dir &lt;string&gt; -dfile &lt;string&gt;</code></pre>

The meaning of each parameter is detailed below:

* ``-alpha <double>``: Value of &alpha;, the parameter for the symmetric Dirichlet prior on &theta; (distributions over topics). Note that the concentration parameter actually used is &alpha;/T. Default value: 1.0.
* ``-beta <double>``: Value of &beta;, the concentration parameter for the symmetric Dirichlet prior on &phi; (distributions over words). Default value: 0.01.
* ``-delta <double>``: Value of &delta;, the parameter for the symmetric Dirichlet prior on &eta; (distributions over interaction recipients). Default value: 1.0.
* ``-gamma <double>``: Value of &gamma;, the parameter for the symmetric Dirichlet prior on &pi; (distributions over communities). Default value: 1.0.
* ``-ntopics <int>``: Number of topics (T). Default value: 10.
* ``-ncomms <int>``: Number of communities (C). Default value: 2.
* ``-niters <int>``: Number of iterations to perform for each chain. Default value: 1000.
* ``-burnin <int>``: Number of iterations before starting collecting samples. Default value: 500.
* ``-lag <int>``: Number of iterations between samples to collect. Default value: 50.
* ``-hypsamp``: Indicates that hyperparameters should be updated (using the auxiliary variable sampling technique) instead of being kept constant. Default value: true.
* ``-nchains <int>``: Number of chains (independent executions of the program, each with a different random initialization) to perform. Default value: 1.
* ``-savestep <int>``: Number of iterations (after burnin) between samples to save in the output files. If the savestep is greater than (niters - burnin), then only one sample (the sample for the last iteration) will be saved for each chain. Default value: 500.
* ``-topwords <int>``: Number of top words (most probable words for each word distribution) to output.
* ``-dir <string>``: Path to the directory containing the data file, and where the output files will be saved.
* ``-dfile <string>``: Name of the data file.

**Example:**
<pre><code>$ java -jar "bin/sn-lda.jar" -alpha 1 -beta 0.01 -delta 1 -gamma 1 -ntopics 15 -ncomms 2 -niters 1000 -burnin 500 -lag 50 -hypsamp -nchains 1 -savestep 500 -topwords 20 -dir "data/midterms" -dfile "midterms.dat"</code></pre>

#### __Output files__

The execution of the collapsed Gibbs sampler for SN-LDA outputs the following files for each savestep (corresponding to a saved model):

* **&lt;model name&gt;.others**: This file contains the value of the hyperparameters (*alpha*, *beta*, *delta*, *gamma*), other parameters (*ntopics*, *ncomms*). It also specifies the number of users in the collection (*nusers*), the size of the vocabulary (*nwords*), as well as the results of the evaluation in terms of various metrics (*perplexity*, *coherence*, *purity*, *inversePurity*, *nmi*, *bCubedPrecision*, *bCubedRecall*, *bCubedF*).
* **&lt;model name&gt;.avgtheta**: This file contains the user-specific distributions over topics &theta;, averaged over the different collected samples.
* **&lt;model name&gt;.avgphi**, **&lt;model name&gt;.avgphi01**, **&lt;model name&gt;.avgphi10**, **&lt;model name&gt;.avgphi11**: These files contain the distributions over background words &phi;<sub>00</sub>, the distributions over viewpoint words &phi;<sub>01</sub>, the distributions over topic words &phi;<sub>10</sub>, and the distributions over viewpoint-topic words &phi;<sub>11</sub>, averaged over the different collected samples. 
* **&lt;model name&gt;.avgeta**: This file contains the community-specific distributions over interaction recipients &eta;, averaged over the different collected samples.
* **&lt;model name&gt;.avgpi**: This file contains the user-specific distributions over communities &pi;, averaged over the different collected samples.
* **&lt;model name&gt;.twords**: This file contains the most probable words in the distributions over topic words &phi;.
* **&lt;model name&gt;.classmap**: This file contains the mapping between the groundtruth label strings (from the input file) and their index used in the output files. The first line corresponds to the number of different viewpoint labels in the collection.
* **&lt;model name&gt;.usermap**: This file contains the mapping between the user ID strings (from the input file) and their index used in the output files. The first line corresponds to the number of different users in the collection.
* **&lt;model name&gt;.wordmap**: This file contains the mapping between the word strings (from the input file) and their index used in the output files. The first line corresponds to the number of different words in the vocabulary.

### __VODUM__

#### __Command line execution__

The collapsed Gibbs sampler for VODUM is run using the following command:
<pre><code>$ java -jar bin/vodum.jar -est [-beta0 &lt;double&gt;] [-beta1 &lt;double&gt;] [-alpha &lt;double&gt;] [-eta &lt;double&gt;] [-ntopics &lt;int&gt;] [-nviews &lt;int&gt;] [-nchains &lt;int&gt;] [-niters &lt;int&gt;] [-savestep &lt;int&gt;] [-topwords &lt;int&gt;] -dir &lt;string&gt; -dfile &lt;string&gt;</code></pre>

The meaning of each parameter is detailed below:

* ``-beta0 <double>``: Value of &beta0;, the concentration parameter for the symmetric Dirichlet prior on &phi;<sub>0</sub> (distributions over topical words). Default value: 0.01.
* ``-beta1 <double>``: Value of &beta1;, the concentration parameter for the symmetric Dirichlet prior on &phi;<sub>1</sub> (distributions over opinion words). Default value: 0.01.
* ``-alpha <double>``: Value of &alpha;, the parameter for the symmetric Dirichlet prior on &theta; (distribution over topics). Note that the concentration parameter actually used is &alpha;/T. Default value: 1.0.
* ``-eta <double>``: Value of &eta;, the parameter for the symmetric Dirichlet prior on &pi; (distribution over viewpoints). Note that the concentration parameter actually used is &eta;/V. Default value: 1.0.
* ``-ntopics <int>``: Number of topics (T). Default value: 10.
* ``-nviews <int>``: Number of viewpoints (V). Default value: 2.
* ``-niters <int>``: Number of iterations to perform for each chain. Default value: 1000.
* ``-burnin <int>``: Number of iterations before starting collecting samples. Default value: 500.
* ``-lag <int>``: Number of iterations between samples to collect. Default value: 50.
* ``-hypsamp``: Indicates that hyperparameters should be updated (using the auxiliary variable sampling technique) instead of being kept constant. Default value: true.
* ``-nchains <int>``: Number of chains (independent executions of the program, each with a different random initialization) to perform. Default value: 1.
* ``-savestep <int>``: Number of iterations (after burnin) between samples to save in the output files. If the savestep is greater than (niters - burnin), then only one sample (the sample for the last iteration) will be saved for each chain. Default value: 500.
* ``-topwords <int>``: Number of top words (most probable words for each word distribution) to output.
* ``-dir <string>``: Path to the directory containing the data file, and where the output files will be saved.
* ``-dfile <string>``: Name of the data file.

**Example:**
<pre><code>$ java -jar "bin/vodum.jar" -beta0 0.01 -beta1 0.01 -alpha 1 -eta 1 -ntopics 15 -nviews 2 -niters 1000 -burnin 500 -lag 50 -hypsamp -nchains 1 -savestep 500 -topwords 20 -dir "data/midterms" -dfile "midterms.dat"</code></pre>

#### __Output files__

The execution of the collapsed Gibbs sampler for SNVDM and SNVDM-GPU outputs the following files for each savestep (corresponding to a saved model):

* **&lt;model name&gt;.others**: This file contains the value of the hyperparameters (*beta0*, *beta1*, *alpha*, *eta*, *ntopics*, *nviews*), other parameters (*ntopics*, *nviews*). It also specifies the number of users in the collection (*nusers*), the size of the vocabulary (*nwords*), as well as the results of the evaluation in terms of various metrics (*perplexity*, *coherence*, *purity*, *inversePurity*, *nmi*, *bCubedPrecision*, *bCubedRecall*, *bCubedF*).
* **&lt;model name&gt;.avgphi0**, **&lt;model name&gt;.avgphi1**: These files contain the distributions over topical words &phi;<sub>0</sub>, and the distributions over opinion words &phi;<sub>1</sub>, averaged over the different collected samples.
* **&lt;model name&gt;.avgtheta**: This file contains the user-specific distributions over topics &theta;, averaged over the different collected samples. 
* **&lt;model name&gt;.avgpi**: This file contains the user-specific distributions over viewpoints &pi;, averaged over the different collected samples.
* **&lt;model name&gt;.twords**: This file contains the most probable words in the distributions over topical words &phi;<sub>0</sub>.
* **&lt;model name&gt;.vtwords**: This file contains the most probable words in the distributions over opinion words &phi;<sub>1</sub>.
* **&lt;model name&gt;.classmap**: This file contains the mapping between the groundtruth label strings (from the input file) and their index used in the output files. The first line corresponds to the number of different viewpoint labels in the collection.
* **&lt;model name&gt;.usermap**: This file contains the mapping between the user ID strings (from the input file) and their index used in the output files. The first line corresponds to the number of different users in the collection.
* **&lt;model name&gt;.wordmap**: This file contains the mapping between the word strings (from the input file) and their index used in the output files. The first line corresponds to the number of different words in the vocabulary.