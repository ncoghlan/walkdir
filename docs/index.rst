WalkDir
=======

.. module:: walkdir
   :synopsis: Tools for iterating over filesystem directories

.. moduleauthor:: Nick Coghlan <ncoghlan@gmail.com>

The standard libary's :func:`os.walk` iterator provides a convenient way to
process the contents of a filesystem directory. This module provides higher
level tools based on the same interface that support filtering, depth
limiting and handling of symlink loops. The module also offers tools that
flatten the :func:`os.walk` API into a simple iteration over filesystem paths.

Walk Iterables
--------------

In this module, ``walk_iter`` refers to any iterable that produces
``path, subdirs, files`` triples sufficiently compatible with those produced
by :func:`os.walk`.

The module is designed so that all purely filtering operations *preserve*
the output of the underlying iterable. This means that named tuples, tuples
containing more than 3 values (such as those produced by :func:`os.fwalk`),
and objects that aren't tuples at all but are still defined such that
``x[0], x[1], x[2] => dirpath, subdirs, files``, can be filtered without being
converted to ordinary 3-tuples.

.. versionchanged:: 0.3
   Objects produced by underlying iterables are now preserved instead of
   being coerced to ordinary 3-tuples by filtering operations

Path Iteration
--------------

Four iterators are provided for iteration over filesystem paths:

.. autofunction:: file_paths

.. autofunction:: dir_paths

.. autofunction:: all_dir_paths

   .. versionadded:: 0.4

.. autofunction:: all_paths

   .. versionchanged:: 0.4
      This function now combines the output of :func:`file_paths` with that
      of :func:`all_dir_paths` (previously it was the combination of
      :func:`file_paths` with :func:`dir_paths`)

Except when the underlying iterable switches to a new root directory, the last
two functions yield subdirectory paths when visiting the parent directory,
rather than when visiting the subdirectory.

For example, given the following directory tree::

    >>> tree test
    test
    ├── file1.txt
    ├── file2.txt
    ├── test2
    │   ├── file1.txt
    │   ├── file2.txt
    │   └── test3
    └── test4
        ├── file1.txt
        └── test5

``all_paths`` will produce::

    >>> from walkdir import filtered_walk, all_paths
    >>> paths = all_paths(filtered_walk('test'))
    >>> print('\n'.join(paths))
    test
    test/file1.txt
    test/file2.txt
    test/test2
    test/test4
    test/test2/file1.txt
    test/test2/file2.txt
    test/test2/test3
    test/test4/file1.txt
    test/test4/test5

``all_dir_paths`` will produce::

    >>> from walkdir import filtered_walk, all_dir_paths
    >>> paths = all_dir_paths(filtered_walk('test'))
    >>> print('\n'.join(paths))
    test
    test/test2
    test/test4
    test/test2/test3
    test/test4/test5

``dir_paths`` will produce::

    >>> from walkdir import filtered_walk, dir_paths
    >>> paths = dir_paths(filtered_walk('test'))
    >>> print('\n'.join(paths))
    test
    test/test2
    test/test2/test3
    test/test4
    test/test4/test5


And ``file_paths`` will produce::

    >>> from walkdir import filtered_walk, file_paths
    >>> paths = file_paths(filtered_walk('test'))
    >>> print('\n'.join(paths))
    test/file1.txt
    test/file2.txt
    test/test2/file1.txt
    test/test2/file2.txt
    test/test4/file1.txt


.. note::
  When used with :func:`min_depth` the output will be produced as multiple
  independent walks of each directory bigger than given *min_depth*.

.. versionchanged:: 0.4
   Subdirectories are now emitted when visiting the parent directory, rather
   than when visiting the subdirectory itself. This means that subdirectories
   may now be emitted without being visited (e.g. subdirectories of directories
   visited by a depth-limited walk, symlinks to subdirectories when not
   following links), and all subdirectories of a given parent directory are
   emitted as a contiguous block, rather than being interleaved with their
   respective file listings.


Directory Walking
-----------------

A convenience API for walking directories with various options is provided:

.. autofunction:: filtered_walk

The individual operations that support the convenience API are exposed using
an :mod:`itertools` style iterator pipeline model:

.. autofunction:: include_dirs

.. autofunction:: include_files

.. autofunction:: exclude_dirs

.. autofunction:: exclude_files

.. autofunction:: limit_depth

.. autofunction:: min_depth

.. autofunction:: handle_symlink_loops


Examples
========

Here are some examples of the module being used to explore the contents
of its own source tree::

    >>> from walkdir import filtered_walk, dir_paths, all_paths, file_paths
    >>> files = file_paths(filtered_walk('.', depth=0,
    ...                    included_files=['*.py', '*.txt', '*.rst']))
    >>> print '\n'.join(files)
    ./setup.py
    ./walkdir.py
    ./NEWS.rst
    ./test_walkdir.py
    ./LICENSE.txt
    ./VERSION.txt
    ./README.txt
    >>> dirs = dir_paths(filtered_walk('.', depth=1, min_depth=1,
    ...                  excluded_dirs=['__pycache__', '.git']))
    >>> print '\n'.join(dirs)
    ./docs
    ./dist
    >>> paths = all_paths(filtered_walk('.', depth=1,
    ...                   included_files=['*.py', '*.txt', '*.rst'],
    ...                   excluded_dirs=['__pycache__', '.git']))
    >>> print '\n'.join(paths)
    .
    ./setup.py
    ./walkdir.py
    ./NEWS.rst
    ./test_walkdir.py
    ./LICENSE.txt
    ./VERSION.txt
    ./README.txt
    ./docs
    ./docs/index.rst
    ./docs/conf.py
    ./dist


Obtaining the Module
====================

This module can be installed directly from the `Python Package Index`_ with
pip_::

    pip install walkdir

Alternatively, you can download and unpack it manually from the `walkdir
PyPI page`_.

There are no operating system or distribution specific versions of this
module - it is a pure Python module that should work on all platforms.

Supported Python versions are 2.6, 2.7 and 3.1+.

.. _Python Package Index: http://pypi.python.org
.. _pip: http://www.pip-installer.org
.. _walkdir pypi page: http://pypi.python.org/pypi/walkdir


Development and Support
-----------------------

WalkDir is developed and maintained on Gitub_, with continuous
integration services provided by `Travis-CI`_.

Problems and suggested improvements can be posted to the `issue tracker`_.

.. _Gitub: https://github.com/ncoghlan/walkdir
.. _Travis-CI: https://travis-ci.org/ncoghlan/walkdir
.. _issue tracker: https://github.com/ncoghlan/walkdir/issues


.. include:: ../NEWS.rst


Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`

