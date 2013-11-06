==================
Session I: Testing
==================

For the rest of the sessions, you'll need an account at http://github.com/
as well as an account at https://readthedocs.org/.

The 'sqer' Python package
-------------------------

We're going to create a Python utility called 'sqer' to give read statistics.

Let's start by creating a 'sqer' library.

Make a directory 'sqer'.

Inside this make another directory 'sqer'.

In 'sqer/sqer' open a file '__init__.py'

From within the top level directory 'sqer' you should be able to do ::

   python -c "import sqer; print sqer"

----

Go into the 'sqer' directory and initialize a git repo::

   git init

Add the sqer/ package directory::

   git add *

   git status

Note the .pyc file -- this is not a source file, but rather a generated file.
Let's remove it from the commit::

   git rm --cached sqer/__init__.pyc

and also ignore it from here on out::

   echo '*.pyc' > .gitignore

Now::

   git status

will not show it as a file, and 'git add' will not add it unless it's
forced.

Next, ::

   git add .gitignore

and commit::

   git commit -am "initial commit"

Now 'git status' should show you only untracked files, no differences.

.. note::

   You can use 'git log' to get a history of commits.

-----

Writing some code (and tests)
-----------------------------

Let's start by writing a function that computes the sum of legit DNA
bases in a sequence record.  The main thing you need to know here is
that each sequence record will come from the `screed
<https://screed.readthedocs.org>`__ utility, which will give us record
objects with a 'name', 'sequence', optional 'accuracy' (for FASTQ),
and optional 'description' (from the FASTA/FASTQ sequence name).

So, put::

   def sum_bp(record):
       return len(record.sequence)

in 'sqer/__init__.py'.

Now, let's add a test. Create a directory 'tests' and put a file
'test_basic.py' in it; this file should contain::

   import sqer

   class FakeRecord(object):
       def __init__(self, sequence, name=''):
           self.sequence = sequence
           self.name = name

   def test_sum_bp():
       r = FakeRecord('ATGC')
       assert sqer.sum_bp(r) == 4

Here, 'FakeRecord' is a stub object that lets you test your code by
faking an object type solely for testing.

Now, run::

   nosetests

You should see::

   .
   ----------------------------------------------------------------------
   Ran 1 test in 0.003s
   
   OK

You can also run 'nosetests -v' to get more verbose output.

Tests pass?  Great, add and commit it with git! ::

   git add tests
   git commit -am "initial tests"

Now, let's add a new function, 'sum_bp_records', to ``sqer/__init__.py``. ::

   def sum_bp_records(records):
       total = 0
       for record in records:
           total += sum_bp(record)

       return total

How shall we test this?  Well, all it expects is an iterable of records:
add this to tests/test_basic.py::

   def test_sum_bp_records():
       rl = [ FakeRecord("A"), FakeRecord("G") ]
       assert sqer.sum_bp_records(rl) == 2

Now run 'nosetests' again -- works? No complaints?

Great, commit it with git::

   git status
   git commit -am "added sum_bp_records and test"

Exercises
~~~~~~~~~

1. Write a test to handle (and ignore) non-ACGT. (Fix the code.)

2. Write a test to verify that lower-case is handled. (Fix the code.)

3. Write a function to calculate the average length of records in a file;
   test it.

Writing a script
----------------

Let's write something to let us use this from the command line.  Put the
following code in ``count-read-bp.py``::

   #! /usr/bin/env python
   import argparse
   import screed
   import sqer
   
   def main():
       parser = argparse.ArgumentParser()
       parser.add_argument('filenames', nargs='+')

       args = parser.parse_args()
       
       total = 0
       for filename in args.filenames:
       	   records = screed.open(filename)
       	   total += sqer.sum_bp_records(records)

       print '%d bp total' % total

   if __name__ == '__main__':
       main()

Next, 'chmod +x count-read-bp.py'.  This makes UNIX aware that it's an
executable file.
	   
Try running it::

    ./count-read-bp.py

Note the friendly error message! Note that you can use '-h', too.

-----

How do we test this??

Put::

    >a
    ATCG
    >b
    GCTA

in a file 'reads.fa'.  Then::

    ./count-read-bp.py reads.fa

You should see '8 bp total'.  Great!

Commit::

   git add count-read-bp.py reads.fa
   git commit -am "command-line script count-read-bp, plus test data"

Check with 'git status'. Do you have editor remainder files (like ~ files
from using emacs)?  Add them to .gitignore and commit the changes.

Testing command line scripts
----------------------------

Put this in a file 'tests/test_scripts.py'::

    import subprocess
    import os
    thisdir = os.path.dirname(__file__)
    thisdir = os.path.normpath(thisdir)

    sqerdir = os.path.join(thisdir, '../')
    sqerdir = os.path.normpath(sqerdir)


    def test_count_reads():
        scriptpath = os.path.join(sqerdir, 'count-read-bp.py')
        datapath = os.path.join(sqerdir, 'reads.fa')

        p = subprocess.Popen([scriptpath, datapath],
                             stdout=subprocess.PIPE,
                             stderr=subprocess.PIPE)
        (out, err) = p.communicate()

        assert p.returncode == 0
        assert "8 bp total" in out, out

Now run 'nosetests' -- what does it say?

Add and commit::

   git add tests/test_scripts.py
   git commit -am "added test for the count-read-bp.py script"

Regression tests with command line scripts
------------------------------------------

Grab some data from somewhere (e.g. 25k.fq.gz from training files) and
put it in ``test-reads.fq``.  You can subset the 25k.fq.gz file if you want::

    gunzip -c 25k.fq.gz | head -400 > test-reads.fq

Add another test to ``sqer/test_scripts.py``::

    def test_count_reads_2():
        scriptpath = os.path.join(sqerdir, 'count-read-bp.py')
        datapath = os.path.join(sqerdir, 'test-reads.fq')
        print thisdir, sqerdir, scriptpath, datapath

        p = subprocess.Popen([scriptpath, datapath],
                             stdout=subprocess.PIPE,
                             stderr=subprocess.PIPE)
        (out, err) = p.communicate()

        assert p.returncode == 0
        assert "8 bp total" in out, out

And now run 'nosetests'.

It should break, right? :)

Fix the last 'assert' code, then rerun; when it all passes, do::

   git add test-reads.fq
   git status

Make sure that only what you think should be there is there; then do::

   git commit -am "added regression test"

Reorganize
----------

Let's put the data files under data/::

   mkdir data
   mv test-reads.fq data
   mv reads.fa data/test-reads.fa

...now, fix the tests!

Exercises
---------

1. Add friendly output to the script, e.g. files opened, # records processed.

2. Add a flag for 'silence'::

      parser.add_argument("-s", dest="silent", type=bool)

   and ::

      if args.silent: ...

Testing summary
---------------

Points to cover:

* any functions named 'test*' in files named 'test*' are executed.

* unit tests are for small bits of code;

* script tests (the first one) are for testing the script API;

* regression tests are for making sure behavior stays the same.
  (We didn't actually count the number of bases in that file, right?
  We just assumed it was counting them right.)

* these three types of tests are for *different purposes* and test different
  things!  Which one is most useful?

* 'print' statements and the like inside the tests are captured, and only
  output upon error.

* assert statements are the way to check things.

Slightly more advanced topics if people are interested:

* what do you do about output files? (temp directories)

* how do you measure if your tests are "good enough"? (code coverage)

Advanced exercises
------------------

4. Write a reservoir sampling algorithm.


----

External resources
==================

* An Introduction to the Nose Testing Framework

  http://ivory.idyll.org/articles/nose-intro.html
