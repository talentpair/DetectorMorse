DetectorMorse
=============

DetectorMorse is a program for sentence boundary detection (henceforth, SBD), also known as sentence segmentation. Consider the following sentence, from the Wall St. Journal portion of the Penn Treebank:

    Rolls-Royce Motor Cars Inc. said it expects its U.S. sales to remain
    steady at about 1,200 cars in 1990.

This sentence contains 4 periods, but only the last denotes a sentence boundary. The first one in `U.S.` is unambiguously part of an acronym, not a sentence boundary; the same is true of expressions like `$12.53`. But the periods at the end of `Inc.` and `U.S.` both could easily denote a sentence boundary. Humans use the local context to determine that neither period denote sentence boundaries (e.g. the selectional properties of the verb _expect_ are not met if there is a sentence bounary immediately after `U.S.`). DetectorMorse uses artisinal, handcrafted contextual features and machine learning to automatically detect sentence boundaries.

SBD is one of the earliest pieces of many natural language processing pipelines. Since errors at this step are likely to propagate, SBD is an important---albeit overlooked---problem in natural language processing.

DetectorMorse has been tested on CPython 3.4 and PyPy3 (2.3.1, corresponding to Python 3.2); the latter is much faster. DetectorMorse depends on the Python module `jsonpickle` to (de)serialize models and uses `nltk`'s `word_tokenize` function during feature extraction. See `requirements.txt` for versions used for testing.

Usage
=====

     usage: detectormorse.py [-h] [-v] [-V] (-t TRAIN | -r READ)
                             (-s SEGMENT | -w WRITE | -e EVALUATE) [-C] [-T T]
     
     DetectorMorse, by Kyle Gorman
     
     optional arguments:
       -h, --help            show this help message and exit
       -v, --verbose         enable verbose output
       -V, --really-verbose  enable even more verbose output
       -t TRAIN, --train TRAIN
                             training data
       -r READ, --read READ  read in serialized model
       -s SEGMENT, --segment SEGMENT
                             segment sentences
       -w WRITE, --write WRITE
                             write out serialized model
       -e EVALUATE, --evaluate EVALUATE
                             evaluate on segmented data
       -C, --nocase          disable case features
       -T T                  # of epochs (default: 20)


Method
======

First, we extract the tokens to the left (L) and right (R) of the period.
If these tokens match a regular expression for American English numbers (including prices, decimals, negatives, etc.), they are merged into a special token `*NUMBER*` (per Kiss & Strunk 2006).

There are two groups of features that are extracted. The first pertains to whether the preceding word is likely to be an abbrevation or not. Intuitively, this lowers the probability that a sentence boundary is present. These features are:

* identity of L (Reynar & Ratnaparkhi 1997)
* does L contain a vowel? (Mikheev 2002)
* does L contain a period? (Grefenstette 1999)
* length of L (Riley 1989)
* case of L (Riley 1989)

The second group pertains directly to whether this is likely to be a sentence boundary (some are repeated from before):

* identity of L (Reynar & Ratnaparkhi 1997)
* is L followed by any quote characters?
* identity of L and R (Reynar & Ratnaparkhi 1997)
* case of R (Riley 1989)
* (quantized) probability of L being final (after Gillick 2009)
* (quantized) probability of R being initial (after Riley 1989)
* (quantized) probability of R being uppercase (after Gillick 2009)

These features are fed into an online classifier (the averaged perceptron; Freund & Schapire 1999) which predicts whether an area of interest contains a sentence boundary.

Exciting extras!
================

I've included a Perl script `untokenize.pl` which attempts to invert the Penn Treebank tokenization process. Tokenization is an inherently "lossy" procedure, so there is no guarantee that the output is exactly how it appeared in the WSJ. But, the rules appear to be correct and produce sane text, and I have used it for all experiments.

References
==========

Y. Freund & R.E. Schapire. 1999. Large margin classification using the perceptron algorithm. _Machine Learning_ 37(3): 277-296.

D. Gillick. 2009. Sentence boundary detection and the problem with the U.S. In _Proc. NAACL-HLT_, pages 241-244.

G. Grefenstette. 1999. Tokenization. In H. van Halteren (ed.), _Syntactic wordclass tagging_, pages 117-133. Dordrecht: Kluwer.

T. Kiss & J. Strunk. 2006. Unsupervised multilingual sentence boundary detection. _Computational Linguistics_ 32(4): 485-525.

A. Mikheev. 2002. Periods, capitalized words, etc. _Computational Linguistics_ 28(3): 289-318.

J.C. Reynar & A. Ratnaparkhi. 1997. A maximum entropy approach to identifying sentence boundaries. In _Proc. 5th Conference on Applied Natural Language Processing_, pages 16-19.

M.D. Riley. 1989. Some applications of tree-based modelling to speech and language indexing. In _Proc. DARPA Speech and Natural Language Workshop_, pages 339-352.