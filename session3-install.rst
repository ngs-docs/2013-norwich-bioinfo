Using virtualenv
================

To create a virtual environment::

   python -m virtualenv ~/env

To activate it::

   . ~/env/bin/activate

Now, even without root, you can do ``pip install`` of whatever packages
you like.

Building a 'setup.py'
=====================

In the sqer directory,

1. grab the latest ez_setup.py from https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py::

   curl -O https://bitbucket.org/pypa/setuptools/raw/19873119647deae8a68e9ed683317b9ee170a8d8/ez_setup.py

2. Put the following in setup.py::

    import ez_setup
    ez_setup.use_setuptools()

    from setuptools import setup

    setup(name="sqer",
          version="0.1",
          packages=['sqer'],
          install_requires=["screed >= 0.7"],
          setup_requires=["nose >= 1.0",],
          scripts=["count-reads.py"],
	  test_suite = 'nose.collector',
    )

3. Put the following in setup.cfg::

   [nosetests]
   verbosity = 2

Now you can do::

   python setup.py test

and::

   python setup.py install

Remember to do::

   git add setup.cfg setup.py
   git commit -am "made it installable"

Note that if you create a .tar.gz, ::

   cd ..
   tar czf /tmp/sqer.tar.gz sqer
   cd sqer

you can now do::

   pip install /tmp/sqer.tar.gz

and this will also work with URLs to the .tar.gz as well as github
files & release links...
   
Building a default/basic 'Makefile'
===================================

Put the following in 'Makefile' in the seqr/ directory::

   all:
	python setup.py build

   install:
	python setup.py install

   clean:
	python setup.py clean
	rm -fr build

   test:
	python setup.py test

Remember to::

   git add Makefile
   git commit -am "added Makefile"

Documentation
=============

We're going to build some docs using Sphinx and reStructuredText.

Do::

   mkdir doc
   cd doc
   sphinx-quickstart

Use default values for everything; specify project name, author, and version.

Now, in the 'doc' directory, do::

   make html

and look at _build/html/index.html

Let's flesh this out a bit -- edit 'index.rst' and add an indented
'details' under Contents, e.g.::

   Contents:

   .. toctree::
      :maxdepth: 2

      details

Now create 'details.rst' to contain::

   ===============
   Project Details
   ===============

   sqer is awesome.

   Important details
   =================

   This where all my documentation goes.

...and run 'make html' again.  Look at _build/html/index.html.

Be sure to do::

   rm -fr _build
   git add *
   git commit -am "added docs"

Now, push this all to github::

   git push origin master

and let's go configure it at http://readthedocs.org/.

