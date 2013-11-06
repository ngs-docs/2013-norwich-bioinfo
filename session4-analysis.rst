==================================================
Session 4: Analysis pipelines and IPython Notebook
==================================================

A slightly more useful sqer script
----------------------------------

Put the following in ``sqer/calc-lengths.py``::

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
          for record in records:
             print len(record.sequence)

   if __name__ == '__main__':
      main()

then ::

   chmod +x calc-lengths.py
   git add calc-lengths.py
   git commit -am "added calc-lengths.py"

Exercises
~~~~~~~~~

1. Write a test for calc-lengths.py!

Write a little analysis pipeline
--------------------------------

Create a directory ``pipeline`` under sqer::

   mkdir pipeline

and copy in the 'trinity-nematostella.fa.gz' file from the training files
into this directory (any FASTA/FASTQ file will do here), gunzip it,
and then rename it to ``assembly.fa``.

Now, create ``pipeline/Makefile`` containing::

   all: lengths.txt

   lengths.txt: assembly.fa
   	../calc-lengths.py assembly.fa > lengths.txt

Now, when you type 'make', it will run your analysis pipeline.
(...pretend that 'calc-lengths.py' takes a long time or something :)

Start up IPython Notebook
-------------------------

From within the pipeline directory, run::

   ipython notebook --pylab=inline

Click on 'New Notebook'.  In this new notebook, enter::

   data = numpy.loadtxt('lengths.txt')
   hist(data, bins=100)
   xlabel('Sequence lengths')
   ylabel('N sequences with that length')
   title('Sequence length spectrum')
   savefig('hist.pdf')

and hit 'Shift-ENTER' to execute.

Voila!

Save the notebook (File... save...)

Now, do (from within the pipeline directory)::

   ls -1 assembly.fa lengths.txt > .gitignore
   git add Makefile .gitignore *.ipynb
   git commit -am "analysis makefile and notebook"

and then::

   git push origin master

Now go find the raw URL to your notebook on github, copy it, and then
paste it in at::

   http://nbviewer.ipython.org

Voila!

Additional IPython resources:

* The ipynb site: http://ipython.org/notebook.html
* A gallery of interesting notebooks: https://github.com/ipython/ipython/wiki/A-gallery-of-interesting-IPython-Notebooks
* The matplotlib gallery: http://matplotlib.org/gallery.html

Note that you can use '%loadpy' in IPython Notebook to grab code from online
and import it into your notebook automagically.
