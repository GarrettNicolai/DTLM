Welcome to DTLM, a semi-supervised seq2seq system for low-resource transduction.

Note that due to submission size constraints, we could not include all the necessary
libraries to make DTLM and M2M.

M2M further requires the tclap library,
while DTLM requires SVMlight, STLPort, and tclap.

To take full advantage of DTLM, you will require 2 additional resources, 
possibly compiled from an unannotated corpus:

A language model file, in the .ARPA format.  For this paper, LMs were
constructed using the CMU language modeling toolkit, but all that should
be necessary is a language model in .ARPA format.

A word list consisting of counts and words, separated by a tab.

ie:

200214	the
192245	a
46030	is


To start, your training file should contain source and target,
separated by a tab.  All individual characters in the file should be 
separated by spaces (_ and | are reserved characters, so please replace them
with something else):

w $ k # z	w a l k e r s

Call m2m-aligner with the following command:

./m2m+ --maxX 1 --maxY 1 --delX --delY -i train.txt -o train.pass1

This will produce the primary alignment:

w|_|$|k|_|#|z|	w|a|l|k|e|r|s|

Replace all pipes with spaces:

sed -i 's/|/ /g' train.pass1

w _ $ k _ # z	w a l k e r s

And run the second pass of the aligner:

./m2m+ --maxX 1 --maxY 1 -i train.pass1 -o train.pass2

w|_:$|k|_:r|z|	w|a:l|k|e:r|s|

Remove all null mergings:

sed -i 's/_://g' train.pass2
sed -i 's/:_//g' train.pass2

w|$|k|r|z|	w|a:l|k|e:r|s|

And run DTLM:

./DTLM --cs 3 --ng 7 --jointMgram 3 --inChar : --outChar ' ' -t test.txt -a test.out -f train.pass2 --mo modelName

Note that the test file should have the source only, separated by :

r:U:n:#:z

If you want to test on another file later, use the second-to-last generated
model (DTLM runs until convergence, so the last model will usually be slightly
worse than the second-to-last).

