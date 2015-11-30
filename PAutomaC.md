# PAutomaC #

[PAutomaC](http://ai.cs.umbc.edu/icgi2012/challenge/Pautomac/) was a machine learning/grammatical inference competition held at [ICGI2012](http://grammarlearning.org). The task was to, from 20,000/100,000 training examples (numerical sequences) generated by an unknown PFA/HMM, infer a mechanism by which to assign probabilities to 1,000 unseen sequences, generated by the same unknown HMM/PFA.

In total, there were 48 separate problems.

The evaluation was done by a perplexity measure between the correct probabilities and the ones assigned by the participant.

In short, the task was as follows:

  * Step 1: from the training examples, infer a PFA/HMM
  * Step 2: assign probabilities to sequences in the test set using this PFA/HMM

<img src='http://treba.googlecode.com/svn/wiki/pautomacexample.png' alt='pautomac workflow' width='600px' />

Of course, the learned structure doesn't have to be a PFA/HMM since we only need to assign probabilities to sequences. However, the best-performing results all used some type of finite state machine.

The three top finishers used mainly a

  * Gibbs sampler
  * EM/Baum-Welch
  * A state-merging algorithm

in their participation (Verwer, et al., 2013).

As **treba** contains these algorithms (and more), the results can be reproduced (roughly, because of a random component in Gibbs sampling/EM). Also, other strategies and parameters can be explored. One can compare the output with PAutomaC competitor results [here](http://ai.cs.umbc.edu/icgi2012/challenge/Pautomac/results.php.htm).

# Running treba on PAutomaC data #

There is a wrapper perl-script to make it easy to run treba on the PAutomaC problems.  The PAutomaC data set `treba-pautomac.tar.gz` in the Downloads directory contains both the script and the data. Running the script presumes access to the PAutomaC files from the tarball, and **treba** installed in the system path.

The perl-wrapper accepts two arguments:

```
./pautomac.perl PAUTOMACPATH PROBLEMNUMBER TREBAARGUMENTS
```

The first argument is the path to the PAutomaC files. The second argument is the training arguments to pass to treba.

After extracting the file `treba-pautomac.tar.gz` and entering the directory `treba-pautomac` you should be able to run the below examples of basic usage, assuming you have treba installed in the system path.

## Example 1 (state merging) ##

For example, running

```
./pautomac.perl pautomac-data 2 --train=merge --alpha=0.3
```

would run **treba** on problem number **2**, using a state-merging algorithm (ALERGIA), with the alpha parameter set to 0.3, and assuming the PAutomaC files lie in `pautomac-data` under the current directory.

The output should look like:

```
Found 502 accessible states
Converting to PFSA: 502 states 19 symbols
loglikelihood=-405656.53709227988

Minimal perplexity (solution): 168.330805339028
Perplexity: 168.419839769882
```

You can compare these results to the perplexity results by the competitors [here](http://ai.cs.umbc.edu/icgi2012/challenge/Pautomac/results.php.htm).

## Example 2 (EM) ##

The following command would run problem 30, using Baum-Welch, assuming a PFA of 12 states:

```
./pautomac.perl pautomac-data 30 --train=bw --initialize=12
```

A typical output would be:

```
iteration 1 loglikelihood=-599534.19805623847 delta: 599534.19805623847
iteration 2 loglikelihood=-462694.378248238 delta: 136839.81980800047
iteration 3 loglikelihood=-448941.92711663974 delta: 13752.451131598267
...
Minimal perplexity (solution): 22.9259853769697
Perplexity: 22.9325256667927
```

## Example 3 (Gibbs sampler) ##

The following would run problem **14** using a Gibbs sampler (with the NVIDIA CUDA GPU code), assuming a 15-state HMM, otherwise with default parameters:

```
./pautomac.perl pautomac-data 14 --train=gs -x 10000 --cuda --initialize=15 --hmm
```

If you don't have CUDA support compiled or don't have an NVIDIA GPU, just drop the --cuda flag. With CUDA, the process should take just a few seconds, and a few minutes without.

A typical output is:

```
Launching CUDA.
10        -624409.37914701249
100       -404375.68352265889
1000      -403576.49450956704
2000      -403030.27397241857
3000      -402871.12854282453
4000      -402725.76046653005
5000      -402631.94331999432
6000      -402582.24548008951
7000      -402569.96502006945
8000      -402550.61233055254
9000      -402535.23549377575
loglikelihood=-402524.65197596449

Minimal perplexity (solution): 116.791881845785
Perplexity: 116.834283964982
```

## PAutomaC data format ##

The wrapper script takes care of format conversions to feed treba the plain data and evaluate the output. However, for the sake of documentation, here is brief documentation of the raw data provided by PAutomaC.

For each of the **48** problems, there are three files.

```
n.pautomac.train
n.pautomac.test
n.pautomac_solution.txt
```

The train file contains the training data with whitespace-separated integers, one training example on each line. The first line in the file contains an indication of how many lines of training data there are in total, and the alphabet size. Also, the first number on each line indicates the length of each example. For example:

```
20000 13          # 20,000 lines, alphabet size 13
3 5 6 7           # length 3 example, sequence=5 6 7
4 11 8 8 7        # ...
3 5 6 7           # ...
...
```

The test files are in the same format, but only contain 1,000 sequences.

The solution files contain the "correct" probabilities for each line corresponding to the test set, with the first line again indicating the number of lines.

```
1000
0.353703205099
1.04065038387e-08
0.000383426911229
1.73723655737e-06
...
```


---


# References #

[PAutomac Web Page](http://ai.cs.umbc.edu/icgi2012/challenge/Pautomac/results.php.htm)

Shibata, C., & Yoshinaka, R. (2012). Marginalizing Out Transition Probabilities for Several Subclasses of PFAs. Journal of Machine Learning Research Workshop and Conference Proceedings, Vol. 21: ICGI 2012: 259-263.

Verwer, S., Eyraud, R., and de la Higuera, C. (2013). PAutomaC: a PFA/HMM learning competition. _Machine Learning_.