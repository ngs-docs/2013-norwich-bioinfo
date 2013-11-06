=======================================
Session 3: Python packages and installs
=======================================

Using virtualenv
================

To create a virtual environment::

   python -m virtualenv ~/env

To activate it as your default Python environment::

   . ~/env/bin/activate

Now, even without root, you can do ``pip install`` of whatever packages
you like.

To deactivate it, ::

   deactivate

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

to run the tests, and::

   python setup.py install

This will install 'sqer' so that (a) it's importable from anywhere, ::

   python -c "import sqer"

and (b) the script(s) are in your path so that::

   count-read-bp.py data/test-reads.fa

should work from anywhere.

Remember to add and commit to git::

   git add setup.cfg setup.py
   git commit -am "added install configuration"

Note that if you create a .tar.gz, ::

   cd ..
   tar czf /tmp/sqer.tar.gz sqer
   cd sqer

you can now do::

   pip install /tmp/sqer.tar.gz

and this will also work with URLs to the .tar.gz as well as github
files & release links...

One final comments: 'git status' will show you that the directory is
getting messy.  Add::

   *.egg
   *.egg-info
   build

to .gitignore, and then commit::

   git commit -am "updated gitignore with setup.py detritus"

It's probably time to do a 'git push origin master' too!
   
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

.. note::

   'make' is picky about tabs vs spaces -- the lines after the ':' need
   to be indented with tabs to work properly.

This will now let us do 'make' (which will execute the first target,
'all'); 'make install'; 'make clean'; and 'make test'.  These will
do the obvious things.

The important thing here is that all of these are *standard* make
commands.  If I see a Makefile in a repository, then I assume that
it's got the commands above.  Convention, convention, convention!

Remember to::

   git add Makefile
   git commit -am "added Makefile"

Documentation
=============

We're going to build some docs using `Sphinx <http://sphinx-doc.org/>`__ and
`reStructuredText <http://docutils.sourceforge.net/rst.html>`__.

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

And also add a rule to the top-level Makefile::

   doc:
	cd doc && make html

(and git add/commit the Makefile changes.)

Now, push this all to github::

   git push origin master

and let's go configure it at http://readthedocs.org/.

Reminder: under your github project, settings, service hooks, enable
the 'readthedocs' service hook.
