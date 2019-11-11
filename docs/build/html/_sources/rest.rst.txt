.. -*- coding: utf-8 -*-
.. _restructured-text-label:

======================
How to edit these docs
======================

You'll need a GitHub account and push permissions for the ``OMaralab/sushi`` repo.

Conda environment
=================

Make a new conda environment::

    conda create --name docs
    conda activate docs

Install sphinx, sphinx_rtd_theme, and sphinx-autobuild::

    conda install sphinx
    pip install sphinx_rtd_theme
    pip install sphinx-autobuild

Download this repo::

    git clone git@github.com:OMaraLab/sushi.git
    cd sushi/docs

Building
========

Link pages to ``index.rst``. You can write in Markdown or RestructuredText, I think. RestructuredText is more flexible while Markdown is much quicker. Check how your docs look with::

    python -m sphinx_autobuild source build

To push back to GitHub, you need to make the files. I usually remove the ``build`` directory first to ensure everything is refreshed. ::

    rm build
    make github
    git add .
    git commit -m 'updated'
    git push origin master

